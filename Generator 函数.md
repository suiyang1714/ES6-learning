## 引子
>　最早接触这个 *Generator 函数* 应该是写公众号的时候看大佬的代码运用了 *Generator 函数* ，也去学习了一发，一知半解就过去了，整理下笔记再去了翻翻demo，看看能看懂了吗...   
前面在学习 *Iterator* 的时候也大体了解下了 *Generator 函数*，结合之前的理解就是执行 *Generator 函数* 就会返回一个遍历器对象，随时可以调用。

## 简介

### 基本概念
Generator 函数是 ES6 提供的一种异步编程解决方案，语法行为与传统函数完全不同。

Generator 函数有多种理解角度。从语法上，首先可以把它理解成，Generator 函数是一个状态机，封装了多个内部状态。

执行 Generator 函数会返回一个遍历器对象，也就是说，Generator 函数除了状态机，还是一个遍历器对象生成函数。返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。

形式上，Generator 函数是一个普通函数，但是有两个特征。一是，function关键字与函数名之间有一个星号；二是，函数体内部使用yield表达式，定义不同的内部状态（yield在英语里的意思就是“产出”）。

```
 //生成器函数，ES6内部实现了迭代器功能，你要做的只是使用yield来迭代输出。
    function *createIterator() {
      yield 1;
      yield 2;
      yield 3;
    }
    const a = createIterator();
    console.log(a.next()); //{value: 1, done: false}
    console.log(a.next()); //{value: 2, done: false}
    console.log(a.next()); //{value: 3, done: false}
    console.log(a.next()); //{value: undefined, done: true}
```
每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield表达式（或return语句）为止。换言之，Generator 函数是分段执行的，yield表达式是暂停执行的标记，而next方法可以恢复执行。

### yield 语句

由于 Generator 函数返回的遍历器对象，只有调用next方法才会遍历下一个内部状态，所以其实提供了一种可以暂停执行的函数。yield表达式就是暂停标志。

遍历器对象的next方法的运行逻辑如下。

（1）遇到yield表达式，就暂停执行后面的操作，并将紧跟在yield后面的那个表达式的值，作为返回的对象的value属性值。

（2）下一次调用next方法时，再继续往下执行，直到遇到下一个yield表达式。

（3）如果没有再遇到新的yield表达式，就一直运行到函数结束，直到return语句为止，并将return语句后面的表达式的值，作为返回的对象的value属性值。

（4）如果该函数没有return语句，则返回的对象的value属性值为undefined。

需要注意的是，yield表达式后面的表达式，只有当调用next方法、内部指针指向该语句时才会执行，因此等于为 JavaScript 提供了手动的“惰性求值”（Lazy Evaluation）的语法功能。


```
function* gen() {
  yield  123 + 456;
}
```
上面代码中，yield后面的表达式123 + 456，不会立即求值，只会在next方法将指针移到这一句时，才会求值。

* yield 语句和return语句的区别主要是：一个函数里面只能执行一次 return 语句，但是可以执行多次 yield 语句。  
* 当执行了 return 语句之后，其后面的 yield 语句将不会在执行。
* yield 语句放在普通函数会报错。
* yeild 语句用在一个表达式中，必须放在圆括号内。


## next 方法的参数

yield表达式本身没有返回值，或者说总是返回undefined。next方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值。

这个功能有很重要的语法意义。Generator 函数从暂停状态到恢复运行，它的上下文状态（context）是不变的。通过next方法的参数，就有办法在 Generator 函数开始运行之后，继续向函数体内部注入值。也就是说，可以在 Generator 函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。


```
function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}

var a = foo(5);
a.next() // Object{value:6, done:false}
a.next() // Object{value:NaN, done:false}
a.next() // Object{value:NaN, done:true}

var b = foo(5);
b.next() // { value:6, done:false }
b.next(12) // { value:8, done:false }
b.next(13) // { value:42, done:true }
```

## for...of 循环

for...of循环可以自动遍历 Generator 函数时生成的Iterator对象，且此时不再需要调用next方法。


```
function *foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```

需要注意的是，一旦next方法的返回对象的done属性为true，即返回的是 return 语句，循环便将结束，且不包含该返回的对象。

除了for...of循环以外，扩展运算符（...）、解构赋值和Array.from方法内部调用的，都是遍历器接口。这意味着，它们都可以将 Generator 函数返回的 Iterator 对象，作为参数。


```
function* numbers () {
  yield 1
  yield 2
  return 3
  yield 4
}

// 扩展运算符
[...numbers()] // [1, 2]

// Array.from 方法
Array.from(numbers()) // [1, 2]

// 解构赋值
let [x, y] = numbers();
x // 1
y // 2

// for...of 循环
for (let n of numbers()) {
  console.log(n)
}
// 1
// 2
```
## Generator.prototype.return()

Generator函数返回的遍历器对象，还有一个return方法，可以返回给定的值，并且终结遍历Generator函数。


```
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next()        // { value: 1, done: false }
g.return('foo') // { value: "foo", done: true }
g.next()        // { value: undefined, done: true }
```

如果return方法调用时，不提供参数，则返回值的value属性为undefined。

## yield* 语句

如果在 Generator 函数内部，调用另一个 Generator 函数，默认情况下是没有效果的。

这个就需要用到yield*表达式，用来在一个 Generator 函数里面执行另一个 Generator 函数。


```
function* bar() {
  yield 'x';
  yield* foo();
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  yield 'a';
  yield 'b';
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  for (let v of foo()) {
    yield v;
  }
  yield 'y';
}

for (let v of bar()){
  console.log(v);
}
// "x"
// "a"
// "b"
// "y"
```


yield*后面的 Generator 函数 ，等同于在 Generator 函数内部，部署一个for...of循环。


```
function* concat(iter1, iter2) {
  yield* iter1;
  yield* iter2;
}

// 等同于

function* concat(iter1, iter2) {
  for (var value of iter1) {
    yield value;
  }
  for (var value of iter2) {
    yield value;
  }
}
```
## 作为对象属性的 Generator函数
如果一个对象的属性是 Generator 函数，可以简写成下面的形式。

```
let obj = {
  * myGeneratorMethod() {
    ···
  }
};

//等价于
let obj = {
  * myGeneratorMethod() {
    ···
  }
};
```
