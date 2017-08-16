## 引子 ##

> 对于 *Iterator* 这个单词我是真的不陌生了，在之前几章只要涉及到遍历基本都会提一嘴 “ *Iterator接口 ...详见xx章节* ” ，对于这个概念我先去查了一下，只知道是一个对象，有这个接口的数据结构的都可以遍历，没有的统统不能遍历...一直以来是这么个印象，好不容易看到这一章节，认真学习一蛤，还真没看懂...   
> 试图把笔记做的自己下次看的时候能够想起更多东西。

## Iterator 的概念 ##
### 《 ES6 标准入门 》 的概念 ##
阮老师的书中写的很明确：

*JavaScript* 原有的表示“集合”的数据结构，主要是数组（*Array*）和对象（*Object*），*ES6* 又添加了 *Map* 和 *Set* 。这样就有了四种数据集合，用户还可以组合使用它们，定义自己的数据结构，比如数组的成员是 *Map* ，*Map* 的成员是对象。这样就需要一种统一的接口机制，来处理所有不同的数据结构。

 **遍历器（*Iterator*）就是这样一种机制。它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署 *Iterator* 接口，就可以完成遍历操作（即依次处理该数据结构的所有成员）。**   
 
*Iterator* 的作用有三个：   
* 一是为各种数据结构，提供一个统一的、简便的访问接口；
* 二是使得数据结构的成员能够按某种次序排列；
* 三是ES6创造了一种新的遍历命令 *for...of* 循环，*Iterator* 接口主要供 *for...of* 消费。

*Iterator* 的遍历过程是这样的。

（1）创建一个指针对象，指向当前数据结构的起始位置。也就是说，遍历器对象本质上，就是一个指针对象。

（2）第一次调用指针对象的 *next* 方法，可以将指针指向数据结构的第一个成员。

（3）第二次调用指针对象的 *next* 方法，指针就指向数据结构的第二个成员。

（4）不断调用指针对象的 *next* 方法，直到它指向数据结构的结束位置。

通过看上面的概念，还是很抽象，我根本没有意识到这个它到底是什么的东西存在的。只知道它本身是个特殊对象，作用就是遍历当前的数据结构。当看到这个接口模拟代码时，有种明白了的赶脚：

```
function createIterator(items) {
  var i = 0;
  return {
    next() {
      var done = (i >= items.length); // 判断i是否小于遍历的对象长度。
      var value = !done ? items[i++] : undefined; //如果done为false，设置value为当前遍历的值。
      return {
        done,
        value
      }
    }
  }
}
const a = createIterator([1, 2, 3]);

//该方法返回的最终是一个对象，包含value、done属性。
console.log(a.next()); //{value: 1, done: false}
console.log(a.next()); //{value: 2, done: false}
console.log(a.next()); //{value: 3, done: false}
console.log(a.next()); //{value: undefined, done: true}
```
总结一下应该是：*Iterator* 是一种特殊对象，每一个 *Iterator* 对象都有一个 *next()* ，该方法返回一个对象，包括 *value* 和 *done* 属性。

## 默认 Iterator 接口 ##
大体了解 *Iterator* 是什么东西，还不了解要干什么。这个问题前面也给出了解答—— *Iterator* 接口主要供for...of消费。

回顾刚刚的知识串起来：*Iterator* 接口的目的，就是为所有数据结构，提供了一种统一的访问机制，即 *for...of* 循环。当使用 *for...of* 循环遍历某种数据结构时，该循环会自动去寻找 *Iterator* 接口。

> ES6 规定，默认的 *Iterator* 接口部署在数据结构的 *Symbol.iterator* 属性，或者说，一个数据结构只要具有 *Symbol.iterator* 属性，就可以认为是“可遍历的”（*iterable*）。

这里又蹦出了 *Symbol.iterator* ，就在上一篇笔记虽然没有详细说，但也提到了，本身是内置的 *Symbol* 值。其说明为：
> 对象的Symbol.iterator属性，指向该对象的默认遍历器方法。对象进行for...of循环时，会调用Symbol.iterator方法，返回该对象的默认遍历器。

前面也看到过，在 *ES6* 中，有三类数据结构原生具备 *Iterator* 接口：**数组、某些类似数组的对象，以及 *Set* 和 *Map* 结构。**

```
let arr = ['a', 'b', 'c'];
let iter = arr[Symbol.iterator]();

iter.next() // { value: 'a', done: false }
iter.next() // { value: 'b', done: false }
iter.next() // { value: 'c', done: false }
iter.next() // { value: undefined, done: true }
```

简单的解释一下就是变量arr是一个数组，原生就具有遍历器接口，部署在 *arr* 的  *Symbol.iterator* 属性上面。所以，调用这个属性，就得到遍历器对象。

> 对于原生部署 *Iterator* 接口的数据结构，不用自己写遍历器生成函数，*for...of* 循环会自动遍历它们。除此之外，其他数据结构（主要是对象）的 *Iterator* 接口，都需要自己在 *Symbol.iterator* 属性上面部署，这样才会被 *for...of* 循环遍历。

> 对象（Object）之所以没有默认部署 *Iterator* 接口，是因为对象的哪个属性先遍历，哪个属性后遍历是不确定的，需要开发者手动指定。本质上，遍历器是一种线性处理，对于任何非线性的数据结构，部署遍历器接口，就等于部署一种线性转换。

对于对象没有原生部署 *Iterator* 接口，但是要能够被 *for...of* 循环调用 *Iterator* 接口，这就必须自己在 *Symbol.iterator* 的属性上部署遍历器生成方法（原型链上的对象具有该方法也可）。

```
let obj = {
  data: [ 'hello', 'world' ],
  [Symbol.iterator]() {
    const self = this;
    let index = 0;
    return {
      next() {
        if (index < self.data.length) {
          return {
            value: self.data[index++],
            done: false
          };
        } else {
          return { value: undefined, done: true };
        }
      }
    };
  }
};
```

## 调用 Iterator 接口的场合 ##

### 解构赋值 ###
对数组和 Set 结构进行解构赋值时，会默认调用 *Symbol.iterator* 方法。

```
let set = new Set().add('a').add('b').add('c');

let [x,y] = set;
// x='a'; y='b'

let [first, ...rest] = set;
// first='a'; rest=['b','c'];
```
### 扩展运算符 ###

扩展运算符（...）也会调用默认的 Iterator 接口。


```
// 例一
var str = 'hello';
[...str] //  ['h','e','l','l','o']

// 例二
let arr = ['b', 'c'];
['a', ...arr, 'd']
// ['a', 'b', 'c', 'd']
```
实际上，这提供了一种简便机制，可以将任何部署了 *Iterator* 接口的数据结构，转为数组。也就是说，只要某个数据结构部署了 *Iterator* 接口，就可以对它使用扩展运算符，将其转为数组。

### yield* ###

*yield** 后面跟的是一个可遍历的结构，它会调用该结构的遍历器接口。

```
let generator = function* () {
  yield 1;
  yield* [2,3,4];
  yield 5;
};

var iterator = generator();

iterator.next() // { value: 1, done: false }
iterator.next() // { value: 2, done: false }
iterator.next() // { value: 3, done: false }
iterator.next() // { value: 4, done: false }
iterator.next() // { value: 5, done: false }
iterator.next() // { value: undefined, done: true }
```

### 其他场合 ###

由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口。下面是一些例子。
* for...of
* Array.form()
* Map()、Set()、WeakMap()、WeakSet()（比如 *new Map([["a",1],["b",2]])*）
* Promise.all()
* Promise.race()

## 字符串接口 ##
字符串是一个类似数组的对象，也原生具有 Iterator 接口。

```
var someString = "hi";
typeof someString[Symbol.iterator]
// "function"

var iterator = someString[Symbol.iterator]();

iterator.next()  // { value: "h", done: false }
iterator.next()  // { value: "i", done: false }
iterator.next()  // { value: undefined, done: true }

```

## Iterator接口与Generator函数 ##

最简单实现 *Symbol.iterator* 方法应该还是属于 生成器（Generator函数）
```
var myIterable = {};

myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};
[...myIterable] // [1, 2, 3]

// 或者采用下面的简洁写法

let obj = {
  * [Symbol.iterator]() {
    yield 'hello';
    yield 'world';
  }
};

for (let x of obj) {
  console.log(x);
}
// hello
// world
```
## 遍历器对象的return()，throw() ##
遍历器对象除了具有next方法，还可以具有return方法和throw方法。如果你自己写遍历器对象生成函数，那么next方法是必须部署的，return方法和throw方法是否部署是可选的。


```
function readLinesSync(file) {
  return {
    next() {
      return { done: false };
    },
    return() {
      file.close();
      return { done: true };
    },
  };
}
```
注意，return方法必须返回一个对象，这是 Generator 规格决定的。

throw方法主要是配合 Generator 函数使用，一般的遍历器对象用不到这个方法。

## for...of 循环 ## 

到了这里对于 *for...of* 循环的作用很明确了，但还是得具体有个概念。

> 一个数据结构只要部署了 *Symbol.iterator* 属性，就被视为具有 *iterator* 接口，就可以用 *for...of* 循环遍历它的成员。也就是说，*for...of* 循环内部调用的是数据结构的 *Symbol.iterator* 方法。

*for...of* 循环可以使用的范围包括数组、Set 和 Map 结构、某些类似数组的对象（比如 *arguments* 对象、*DOM NodeList* 对象）、后文的 *Generator* 对象，以及字符串。

此处省略1W字...

### 与其他遍历方法的比较 ###

以数组为例，JavaScript 提供多种遍历语法。最原始的写法就是for循环。


```
for (var index = 0; index < myArray.length; index++) {
  console.log(myArray[index]);
}
```
这种写法比较麻烦，因此数组提供内置的forEach方法。

```
myArray.forEach(function (value) {
  console.log(value);
});
```

这种写法的问题在于，无法中途跳出forEach循环，break命令或return命令都不能奏效。


for...in循环可以遍历数组的键名。

```
for (var index in myArray) {
  console.log(myArray[index]);
}
```

for...in循环有几个缺点。

* 数组的键名是数字，但是for...in循环是以字符串作为键名“0”、“1”、“2”等等。
* for...in循环不仅遍历数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键。
* 某些情况下，for...in循环会以任意顺序遍历键名。

总之，for...in循环主要是为遍历对象而设计的，不适用于遍历数组。

for...of循环相比上面几种做法，有一些显著的优点。


```
for (let value of myArray) {
  console.log(value);
}
```

* 有着同for...in一样的简洁语法，但是没有for...in那些缺点。
* 不同于forEach方法，它可以与break、continue和return配合使用。
* 提供了遍历所有数据结构的统一操作接口。

---
### 总结 ###

知道 *for...of* 的内部原理： *for...of*  要求被遍历的对象实现特定的方法。所有的 Array、Map 和 Set 对象都有一个共性，那就是他们都实现了一个遍历器（iterator）方法。 对象的 *Symbol.iterator* 属性，指向该对象的默认遍历器方法。没有该默认接口的对象，可以通过在 *Symbol.iterator* 属性上部署遍历器生成该方法。

