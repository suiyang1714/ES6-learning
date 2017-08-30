## 引子

> 新增两个数据类型Set和Map，他们能干啥，有啥特点？不知道.。。

## Set ##

### 基本用法

Set 是ES6 提供的新的数据类型，它类似于数组，但是成员的值都是唯一的，没有重复的值。  

Set 本身是一个构造函数，用来生成 Set 数据结构。


```
var s = new Set();

[2,3,4,5,4,5,2,2].map(x=>s.add(x))

for (i of s) { console.log(i) }

// 2,3,4,5

function dedupe(array) {
  return Array.from(new Set(array));
}

dedupe([1, 1, 2, 3]) // [1, 2, 3]

```
上述代码通过 *add* 方法向 *Set* 结构添加成员，结果表明 *Set* 结构不会添加重复的值，也是一种去重的方法。

*Set* 函数也可以接受一个数组(或类似数组的对象) 作为参数，用于初始化。

向 *Set* 加入值时不会发生类型转换，所以 5 和 “5” 是两个不同的值。*Set* 内部判断两个值是否不同时的算法类似精确相等运算符（===）。这意味着，两个对象总是不相等的。主要的区别是NaN等于自身，而精确相等运算符认为NaN不等于自身。**在Set内部，两个NaN是相等。**

### Set 实例的属性和方法
Set结构的实例有以下属性：

* *Set.prototype.constructor* ：构造函数，默认就是 *Set* 函数。
* *Set.prototype.size* ：返回 *Set* 实例的成员总数。

*Set* 实例的方法分为两大类：操作方法（用于操作数据）和遍历方法（用于遍历成员）。下面先介绍四个操作方法。

* *add(value)* ：添加某个值，返回Set结构本身。
* *delete(value)* ：删除某个值，返回一个布尔值，表示删除是否成功。
* *has(value)* ：返回一个布尔值，表示该值是否为Set的成员。
* *clear()* ：清除所有成员，没有返回值。


```
    let set = new Set();
    set.add('haha');
    set.add(Symbol('haha'));
    
    console.log(set.size); //2
    
    console.log(set); 
    Set(2) {"haha", Symbol(haha)}
        size:(...)
        __proto__:Set
        [[Entries]]:Array(2)
            0:"haha"
            1:Symbol(haha)
        length:2
        
    console.log(set.has('haha')) // true
```
*Set* 像数组，又像一个对象，但又不完全是。

### 遍历操作

Set结构的实例有四个遍历方法，可以用于遍历成员。

* *keys()* ：返回一个键名的遍历器
* *values()* ：返回一个键值的遍历器
* *entries()* ：返回一个键值对的遍历器
* *forEach()* ：使用回调函数遍历每个成员

key方法、value方法、entries方法返回的都是遍历器对象。由于Set结构没有键名，只有键值（或者说键名和键值是同一个值），所以key方法和value方法的行为完全一致。

### 迭代Set
使用for of来遍历：

```
  for(let value of sets) {
      console.log(value);
    }
    //"haha"
    //Symbol(haha)
    
    //如果你需要key，则使用下面这种方法
    for(let [key, value] of sets.entries()) {
      console.log(value, key);
    } 
    //"haha" "haha"
    //Symbol(haha) Symbol(haha)
```

forEach操作Set：Set本身没有key，而forEach方法中的key被设置成了元素本身。

```
sets.forEach((value, key) => {
      console.log(value, key);
    });
    //"haha" "haha"
    //Symbol(haha) Symbol(haha)
    
    sets.forEach((value, key) => {
      console.log(Object.is(value, key));
    }); 
    //true true
```
### Set和Array的转换
Set和数组太像了，Set集合的特点是没有key，没有下标，只有size和原型以及一个可迭代的不重复元素的类数组。既然这样，我们就可以把一个Set集合转换成数组，也可以把数组转换成Set。


```
 //数组转换成Set
    const arr = [1, 2, 2, '3', '3']
    let set = new Set(arr);
    console.log(set) // Set(3) {1, 2, "3"}

    //Set转换成数组
    let set = new Set();
    set.add(1);
    set.add('2');
    console.log(Array.from(set)) // (2) [1, "2"]
```
使用Set集合的不可重复性来处理。经测试只能去重下面3种类型的数据。
```
   const arr = [1, 1, 'haha', 'haha', null, null]
    let set = new Set(arr);
    console.log(Array.from(set)) // [1, 'haha', null]
    console.log([...set]) // [1, 'haha', null]
```

## WeakSet

Set集合本身是强引用，只要new Set()实例化的引用存在，就不释放内存，这样一刀切肯定不好啊，比如你定义了一个DOM元素的Set集合，然后在某个js中引用了该实例，但是当页面关闭或者跳转时，你希望该引用应立即释放内存，Set不听话，那好，你还可以使用 Weak Set

### 语法
```
new WeakSet([iterable]);
```
## 和Set的区别：
1、**WeakSet 对象中只能存放对象值**, 不能存放原始值, 而 Set 对象都可以.

2、WeakSet 对象中存储的对象值都是被弱引用的, 如果没有其他的变量或属性引用这个对象值, 则这个对象值会被当成垃圾回收掉. 正因为这样, **WeakSet 对象是无法被枚举的**, 没有办法拿到它包含的所有元素.

### 使用


```
 let set = new WeakSet();
    const class_1 = {}, class_2 = {};
    set.add(class_1);
    set.add(class_2);
    console.log(set) // WeakSet {Object {}, Object {}}
    console.log(set.has(class_1)) // true
    console.log(set.has(class_2)) // true
```

## Map

### Map 结构的目的和基本用法

Map是存储许多键值对的有序列表，key和value支持所有数据类型。

如果说Set像数组，那么Map更像对象。而对象中的key只支持字符串，Map更加强大，支持所有数据类型，不管是数字、字符串、Symbol等。

```
   // 一个空Map集合
    let map = new Map()
    console.log(map)
```

对比Set集合的原型，**Map集合的原型多了set()和get()方法**，注意set()和Set集合不是一个东西。Map没有add，使用set()添加key，value，在Set集合中，使用add()添加value，没有key。

```
let map = new Map();
    map.set('name', 'haha');
    map.set('id', 10);
    console.log(map)
    
    // 输出结果
     Map(2) {"name" => "haha", "id" => 10}
        size:(...)
        __proto__:Map
        [[Entries]]:Array(2)
            0:{"name" => "haha"}
            1:{"id" => 10}
        length:2

    console.log(map.get('id')) // 10
    console.log(map.get('name')) // "haha"
```
如果对同一个键多次赋值，后面的值将覆盖前面的值。  
读取一个未知的键，则返回undefined。

```
const map = new Map();

map.set(['a'], 555);
map.get(['a']) // undefined
```
注意，只有对同一个对象的引用，Map 结构才将其视为同一个键。
上面代码的set和get方法，表面是针对同一个键，但实际上这是两个值，内存地址是不一样的，因此get方法无法读取该键，返回undefined。

```
const map = new Map();

const k1 = ['a'];
const k2 = ['a'];

map
.set(k1, 111)
.set(k2, 222);

map.get(k1) // 111
map.get(k2) // 222
```
由上可知，Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。这就解决了同名属性碰撞（clash）的问题，我们扩展别人的库的时候，如果使用对象作为键名，就不用担心自己的属性与原作者的属性同名。

如果 Map 的键是一个简单类型的值（数字、字符串、布尔值），则只要两个值严格相等，Map 将其视为一个键，比如0和-0就是一个键，布尔值true和字符串true则是两个不同的键。另外，undefined和null也是两个不同的键。虽然NaN不严格相等于自身，但 Map 将其视为同一个键。

### 实例的属性和操作方法

Map结构的实例有以下属性和操作方法。

#### （1）size属性

size属性返回Map结构的成员总数。

#### （2）set(key, value)

set方法设置key所对应的键值，然后返回整个Map结构。如果key已经有值，则键值会被更新，否则就新生成该键。

#### （3）get(key)

get方法读取key对应的键值，如果找不到key，返回undefined。

#### （4）has(key)

has方法返回一个布尔值，表示某个键是否在Map数据结构中。

#### （5）delete(key)

delete方法删除某个键，返回true。如果删除失败，返回false。

#### （6）clear()

clear方法清除所有成员，没有返回值。


### 遍历方法

Map原生提供三个遍历器生成函数和一个遍历方法。

* keys()：返回键名的遍历器。
* values()：返回键值的遍历器。
* entries()：返回所有成员的遍历器。
* forEach()：遍历Map的所有成员。

Map 结构的默认遍历接口是 *Symbol.iterator* 属性，所以可以通过 *for...of* 方法来进行遍历。



### 与其他结构的互相转换

#### Map 转为数组

Map 结构转为数组结构，比较快速的方法是使用扩展运算符（...）。

```
const map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

[...map.keys()]
// [1, 2, 3]

[...map.values()]
// ['one', 'two', 'three']

[...map.entries()]
// [[1,'one'], [2, 'two'], [3, 'three']]

[...map]
// [[1,'one'], [2, 'two'], [3, 'three']]
```
#### 数组转为 Map

将数组传入Map 构造函数里即可。

#### Map 转为对象

如果Map 的所有键都是字符串，则其可以转为对象。

```
function strMapToObj(strMap) {
  let obj = Object.create(null);
  for (let [k,v] of strMap) {
    obj[k] = v;
  }
  return obj;
}

const myMap = new Map()
  .set('yes', true)
  .set('no', false);
strMapToObj(myMap)
// { yes: true, no: false }
```
#### Map 转为 JSON

Map 转为 JSON 要区分两种情况。一种情况是，Map 的键名都是字符串，这时可以选择转为对象 JSON。

```
function strMapToJson(strMap) {
  return JSON.stringify(strMapToObj(strMap));
}

let myMap = new Map().set('yes', true).set('no', false);
strMapToJson(myMap)
// '{"yes":true,"no":false}'
```
另一种情况是，Map 的键名有非字符串，这时可以选择转为数组 JSON。

```
function mapToArrayJson(map) {
  return JSON.stringify([...map]);
}

let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
mapToArrayJson(myMap)
// '[[true,7],[{"foo":3},["abc"]]]'
```

## WeakMap

WeakMap结构与Map结构基本类似，唯一的区别是它只接受对象作为键名（null除外），不接受其他类型的值作为键名，而且键名所指向的对象，不计入垃圾回收机制。



```
 let map = new WeakMap();
    const key = document.querySelector('.header');
    map.set(key, '这是个什么玩意');
    
    map.get(key) // "这是个什么玩意"
    
    //移除该元素
    key.parentNode.removeChild(key);
    key = null;
```
## 总结

Set集合可以用来过滤数组中重复的元素，只能通过has方法检测指定的值是否存在，或者是通过forEach处理每个值。

Weak Set集合存放对象的弱引用，当该对象的其他强引用被清除时，集合中的弱引用也会自动被垃圾回收机制回收，追踪成组的对象是该集合最好的使用方式。

Map集合通过set()添加键值对，通过get()获取键值，各种方法的使用查看文章教程，你可以把它看成是比Object更加强大的对象。

Weak Map集合只支持对象类型的key，所有key都是弱引用，当该对象的其他强引用被清除时，集合中的弱引用也会自动被垃圾回收机制回收，为那些实际使用与生命周期管理分离的对象添加额外信息是非常适合的使用方式。