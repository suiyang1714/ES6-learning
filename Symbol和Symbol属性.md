## 引子 ##

> *Symbol* 是什么？   
> 刚看到这一块并且读完概述之后我还是自问了一句“*Symbol* 是什么？ ”。又读了几遍才有了印象。*Symbol* 本身是一种新的数据类型，它是 *JavaScript* 语言的第7种数据类型。     
> “全文完”  
> 不知道各位看官懂吗？我是懂了...

## 概述 ##

### 由来 ###

ES5 的对象属性名都是字符串，这容易造成属性名的冲突。比如，你使用了一个他人提供的对象，但又想为这个对象添加新的方法（mixin 模式），新方法的名字就有可能与现有方法产生冲突。如果有一种机制，保证每个属性的名字都是独一无二的就好了，这样就从根本上防止属性名的冲突。这就是 ES6 引入Symbol的原因。
### 定义说明 ###

*Symbol* 是一种特殊的、不可变的数据类型，可以作为对象属性的标识符使用。 *Symbol*  对象是一个 *symbol primitive data type* 的隐式对象包装器。

*Symbol* 数据类型是一个原始数据类型。

### Symbol语法格式 ###
Symbol([description]) //description是可选的


```
const name = Symbol();
const name1 = Symbol('sym1');
console.log(name, name1) // Symbol() Symbol(sym1)
```
注意，*Symbol* 函数前不能使用 *new* 命令，否则会报错。这是因为生成的 *Symbol* 是一个原始类型的值，不是对象。也就是说，由于 *Symbol* 值不是对象，所以不能添加属性。基本上，它是一种类似于字符串的数据类型。

```
const name = new Symbol(); 
//Symbol is not a constructor
```
### 作为属性名的 Symbol ###
前面说个每一个Symbol都是独一无二的，都是不相等的，意味着 *Symbol* 值可以作为标识符用于对象的属性名，保证不会出现同名的属性。
```
// 没有参数的情况
var s1 = Symbol();
var s2 = Symbol();

s1 === s2 // false

// 有参数的情况
var s1 = Symbol('foo');
var s2 = Symbol('foo');

s1 === s2 // false
```
在用于属性名的时候有三种写法：

```
var mySymbol = Symbol();

// 第一种写法
var a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
var a = {
  [mySymbol]: 'Hello!'
};

// 第三种写法
var a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });

// 以上写法都得到同样结果
a[mySymbol] // "Hello!"
```

需要注意的是，*Symbol* 值作为对象属性名时，不能用点运算符。

```
var mySymbol = Symbol();
var a = {};

a.mySymbol = 'Hello!';
a[mySymbol] // undefined
a['mySymbol'] // "Hello!"
```
### 属性名的遍历 ###
Symbol 作为属性名，该属性不会出现在 *for...in* 、 *for...of* 循环中，也不会被 *Object.keys()* 、 *Object.getOwnPropertyNames()* 、 *JSON.stringify()* 返回。但是，它也不是私有属性，有一个 *Object.getOwnPropertySymbols* 方法，可以获取指定对象的所有 *Symbol* 属性名。

```
var obj = {};
var a = Symbol('a');
var b = Symbol('b');

obj[a] = 'Hello';
obj[b] = 'World';

var objectSymbols = Object.getOwnPropertySymbols(obj);

objectSymbols
// [Symbol(a), Symbol(b)]
```
### Symbol全局共享 ###
ES6提供了一个注册机制，当你注册Symbol之后，就能在全局共享注册表里面的Symbol。Symbol的注册表和对象表很像，都是key、value结构，只不过这个value是Symbol值。 （key, Symbol） 语法：

```
Symbol.for() //只有一个参数
```
*Symbol.for()* 与 *Symbol()* 这两种写法，都会生成新的 *Symbol* 。它们的区别是，前者会被登记在全局环境中供搜索，后者不会。 *Symbol.for()* 不会每次调用就返回一个新的 *Symbol* 类型的值，而是会先检查给定的*key*是否已经存在，如果不存在才会新建一个值。比如，如果你调用 *Symbol.for("cat")* 30次，每次都会返回同一个 *Symbol* 值，但是调用 *Symbol("cat")* 30次，会返回30个不同的 *Symbol* 值。

```
Symbol.for("bar") === Symbol.for("bar")
// true

Symbol("bar") === Symbol("bar")
// false
```
#### Symbol.keyFor() ####
*Symbol.keyFor()* 方法返回一个已登记的 *Symbol* 类型值的 *Key*

```
var s1 = Symbol.for("foo");
Symbol.keyFor(s1) // "foo"

var s2 = Symbol("foo");
Symbol.keyFor(s2) // undefined
```


### 内置Symbol 值 ###
除了定义自己使用的Symbol值以外，ES6还提供了11个内置的Symbol值，指向语言内部使用的方法。
这些方法用的可能不多，了解下就好

---
> 对于 *Symbol* 的了解可能不需要知道太多，只要知道Symbol如何定义，如何在全局共享，如何在对象中替代key即可应付基本的开发需求了。