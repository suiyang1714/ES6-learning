## 引子 ##

> 最基本的数据类型和引用类型扩展了解完了之后，新的特性、心的概念逐渐接触，学起来也是比较吃力了。虽然用不到但还是要硬着头皮了解下，等哪天用到了最起码有个了解...接下来要了解的是 *Proxy* 和 *Reflect*  这两个新特性，网上流传最多的博文也是像我一样根据阮老师的书来做总结，脱离那些基础知识，实例少的可怜，以至于我懵懵懂懂，不清楚这个在实际开发过程中的灵活运用。  

## Proxy 概述 ##
> 在 *ES6* 中的箭头函数、数组解构、rest 参数等特性一经实现就广为流传的情况下，类似 *Proxy* 这样的新特性却很少见到开发者讨论并且使用，一方面在于浏览器的兼容性，另一方面也在于要想发挥这些特性的优势需要开发者深入地理解其使用场景。

*Proxy* ，见名之意，代理。  
用在这里表示由 *Ta* 来代理某些操作，这些操作可以是“拦截”，“监视”等。所以它的功能作用也有了一个大概的了解：
* 拦截和监视外部对对象的访问；
* 降低函数或类的复杂度；
* 在复杂操作前对操作进行校验或对所需资源尽心管理。


在支持 *Proxy* 的浏览器环境中，*Proxy* 是一个全局对象，可以直接使用。*Proxy(target, handler)* 是一个构造函数，*target*  是被代理的对象，*handlder*  是声明了各类代理操作的对象，最终返回一个代理对象。外界每次通过代理对象访问 target 对象的属性时，就会经过 *handler* 对象，从这个流程来看，代理对象很类似 *middleware*（中间件）。

```
const target = {  
    name: 'Billy Bob',
    age: 15
};
const handler = {  
    get(target, key, proxy) {
        const today = new Date();
        console.log(`GET request made for ${key} at ${today}`);
        return Reflect.get(target, key, proxy);
    }
};
const proxy = new Proxy(target, handler);
proxy.name;
// => "GET request made for name at Thu Jul 21 2016 15:26:20 GMT+0800 (CST)"
// => "Billy Bob"
```
在上面的代码中，我们首先定义了一个被代理的目标对象 *target* ，然后声明了包含所有代理操作的 *handler* 对象，接下来使用 *Proxy(target, handler)* 创建代理对象 *proxy*，此后所有使用 *proxy* 对 *target* 属性的访问都会经过 *handler* 的处理。

## Proxy 实例方法 ##
先来直接看下一完整的可拦截操作列表。

### get () ###

**get(target, propKey, receiver)**

> get() 方法用于拦截某个属性的读取操作。


```
var person = {
  name: "张三"
};

var proxy = new Proxy(person, {
  get: function(target, property) {
    if (property in target) {
      return target[property];
    } else {
      throw new ReferenceError("Property \"" + property + "\" does not exist.");
    }
  }
});

proxy.name // "张三"
proxy.age // 抛出一个错误
```

上面代码表示，如果访问目标对象不存在的属性，会抛出一个错误。如果没有这个拦截函数，访问不存在的属性，只会返回 *undefined*。

### set () ###

> set方法用来拦截某个属性的赋值操作。

假定 *Person* 对象有一个 *age* 属性，该属性应该是一个不大于200的整数，那么可以使用 *Proxy* 保证 *age* 的属性值符合要求。

```
let validator = {
  set: function(obj, prop, value) {
    if (prop === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('The age is not an integer');
      }
      if (value > 200) {
        throw new RangeError('The age seems invalid');
      }
    }

    // 对于age以外的属性，直接保存
    obj[prop] = value;
  }
};

let person = new Proxy({}, validator);

person.age = 100;

person.age // 100
person.age = 'young' // 报错
person.age = 300 // 报错
```
### apply () ###

> apply方法拦截函数的调用、call和apply操作。

*apply* 方法可以接受三个参数，分别是目标对象、目标对象的上下文对象（this）和目标对象的参数数组。

```
var target = function () { return 'I am the target'; };
var handler = {
  apply: function () {
    return 'I am the proxy';
  }
};

var p = new Proxy(target, handler);

p()
// "I am the proxy"
```
### has () ###

> has 方法可以隐藏某些属性，不被 in 操作符发现


```
var handler = {
  has (target, key) {
    if (key[0] === '_') {
      return false;
    }
    return key in target;
  }
};
var target = { _prop: 'foo', prop: 'foo' };
var proxy = new Proxy(target, handler);
'_prop' in proxy // false
```

如果原对象不可配置或者禁止扩展，这时has拦截会报错。


```
var obj = { a: 10 };
Object.preventExtensions(obj);

var p = new Proxy(obj, {
  has: function(target, prop) {
    return false;
  }
});

'a' in p // TypeError is thrown
```
> 该Object.preventExtensions()方法可防止将新属性添加到对象中（即防止对象的未来扩展）。

### construct () ###

> construct方法用于拦截new命令。


```
var p = new Proxy(function () {}, {
  construct: function(target, args) {
    console.log('called: ' + args.join(', '));
    return { value: args[0] * 10 };
  }
});

new p(1).value
// "called: 1"
// 10
```
construct方法返回的必须是一个对象，否则会报错。

### deleteProperty () ###

> deleteProperty方法用于拦截delete操作，如果这个方法抛出错误或者返回false，当前属性就无法被delete命令删除。


```
var handler = {
  deleteProperty (target, key) {
    invariant(key, 'delete');
    return true;
  }
};
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
  }
}

var target = { _prop: 'foo' };
var proxy = new Proxy(target, handler);
delete proxy._prop
// Error: Invalid attempt to delete private "_prop" property
```
### defineProperty () ###

> defineProperty方法拦截了Object.defineProperty操作。


```
var handler = {
  defineProperty (target, key, descriptor) {
    return false;
  }
};
var target = {};
var proxy = new Proxy(target, handler);
proxy.foo = 'bar'
// TypeError: proxy defineProperty handler returned false for property '"foo"'
```
### enumerate () ###

> enumerate 方法用于拦截 *for...in* 循环。注意与 Proxy对象的has 方法区分，后者用于拦截 in 操作符，对 *for...in* 循环无效。

### getOwnPropertyDescriptor ()  ###

> getOwnPropertyDescriptor方法拦截Object.getOwnPropertyDescriptor()，返回一个属性描述对象或者undefined。

> Object.getOwnPropertyDescriptor() 方法返回指定对象上一个自有属性对应的属性描述符。（自有属性指的是直接赋予该对象的属性，不需要从原型链上进行查找的属性）

### getPrototypeOf() ###

> getPrototypeOf方法主要用来拦截获取对象原型。具体来说，拦截下面这些操作。

* Object.prototype.__proto__
* Object.prototype.isPrototypeOf()
* Object.getPrototypeOf()
* Reflect.getPrototypeOf()
* instanceof

### isExtensible() ###

> isExtensible方法拦截Object.isExtensible操作。

> Object.isExtensible() 方法判断一个对象是否是可扩展的（是否可以在它上面添加新的属性）。

### ownKeys() ###

> ownKeys() 方法用于拦截 *Object.keys()* 操作。


```
let target = {
  a: 1,
  b: 2,
  c: 3
};

let handler = {
  ownKeys(target) {
    return ['a'];
  }
};

let proxy = new Proxy(target, handler);

Object.keys(proxy)
// [ 'a' ]
```

### preventExtensions() ###

> preventExtensions方法拦截Object.preventExtensions()。该方法必须返回一个布尔值，否则会被自动转为布尔值。

这个方法有一个限制，只有目标对象不可扩展时（即Object.isExtensible(proxy)为false），proxy.preventExtensions才能返回true，否则会报错。
> Object.preventExtensions() 方法让一个对象变的不可扩展，也就是永远不能再添加新的属性。

### setPrototypeOf() ###

> setPrototypeOf方法主要用来拦截Object.setPrototypeOf方法。

> Object.setPrototypeOf() 方法设置一个指定的对象的原型 ( 即, 内部[[Prototype]]属性）到另一个对象或  null。

## Proxy.revocable() ##

Proxy.revocable方法返回一个可取消的 Proxy 实例。

```
let target = {};
let handler = {};

let {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo // 123

revoke();
proxy.foo // TypeError: Revoked
```
Proxy.revocable方法返回一个对象，该对象的proxy属性是Proxy实例，revoke属性是一个函数，可以取消Proxy实例。上面代码中，当执行revoke函数之后，再访问Proxy实例，就会抛出一个错误。

## Reflect ##
### Reflect 概述 ###
Reflect对象与Proxy对象一样，也是 ES6 为了操作对象而提供的新 API。Reflect对象的设计目的有这样几个。
* 将Object对象的一些明显属于语言内部的方法（比如Object.defineProperty），放到Reflect对象上。现阶段，某些方法同时在Object和Reflect对象上部署，未来的新方法将只部署在Reflect对象上。也就是说，从Reflect对象上可以拿到语言内部的方法。
* 修改某些Object方法的返回结果，让其变得更合理。比如，Object.defineProperty(obj, name, desc)在无法定义属性时，会抛出一个错误，而Reflect.defineProperty(obj, name, desc)则会返回false。
* 让Object操作都变成函数行为。某些Object操作是命令式，比如name in obj和delete obj[name]，而Reflect.has(obj, name)和Reflect.deleteProperty(obj, name)让它们变成了函数行为。
* Reflect对象的方法与Proxy对象的方法一一对应，只要是Proxy对象的方法，就能在Reflect对象上找到对应的方法。这就让Proxy对象可以方便地调用对应的Reflect方法，完成默认行为，作为修改行为的基础。也就是说，不管Proxy怎么修改默认行为，你总可以在Reflect上获取默认行为。

### 静态方法 ###
* Reflect对象一共有13个静态方法。
* Reflect.apply(target,thisArg,args)
* Reflect.construct(target,args)
* Reflect.get(target,name,receiver)
* Reflect.set(target,name,value,receiver)
* Reflect.defineProperty(target,name,desc)
* Reflect.deleteProperty(target,name)
* Reflect.has(target,name)
* Reflect.ownKeys(target)
* Reflect.isExtensible(target)
* Reflect.preventExtensions(target)
* Reflect.getOwnPropertyDescriptor(target, name)
* Reflect.getPrototypeOf(target)
* Reflect.setPrototypeOf(target, prototype)

上面这些方法的作用，大部分与Object对象的同名方法的作用都是相同的，而且它与Proxy对象的方法是一一对应的。下面是对它们的解释。


---
> 走马观花的走了一遍，也具体了解了 *Proxy* 对象和 *Reflect* 对象。 *Proxy*  对象用于定义基本操作的自定义行为 (例如, 属性查找，赋值，枚举，函数调用,等)；而 *Reflect*  是一个内置的对象，它提供可拦截 *JavaScript* 操作的方法。方法与代理处理程序的方法相同。 *Reflect* 不是一个函数对象，因此它是不可构造的。   
然而以后用上的可能性，有待商榷。

### 参考链接
> * [实例解析 ES6 Proxy 使用场景](https://juejin.im/entry/5790b5432e958a0065ec8a0c)
> * [Reflect](https://juejin.im/entry/5878a6cc570c350062111423)
> * [MDN Reflect](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)
> * [MDN Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)