## 引子
> 这一节总结 *Promise* 对象，对于 *Promise* 并不陌生，因为一次需求用到过，所以有点经验，但让我具体说出点什么只能 *say good-bye* 。对它的再一种了解应该是大佬们说的解决回调地狱吧，你问我回调地狱是什么？对不起入门太晚，没见识过。

## Promise 含义

### Promise是什么

Promise的中文意思是承诺，也就是说，JavaScript对你许下一个承诺，会在未来某个时刻兑现承诺。

*Promise* 在 *JavaScript* 语言早有实现，ES6 将其写进了语言标准，统一了写法，并提供了原生的 *Promise 对象*。 

### Promise生命周期

**Promise的生命周期：进行中（pending），已经完成（fulfilled），拒绝（rejected）**   
相比react、vue 的周期还算少的。

* 对象的状态不受外界影响。Promise对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
* 一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从pending变为fulfilled和从pending变为rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 resolved（已定型）。如果改变已经发生了，你再对Promise对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

## 基本用法

ES6 规定，Promise对象是一个构造函数，用来生成Promise实例。

```
var promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```
resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即从 pending 变为 rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。


```
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```
then方法可以接受两个回调函数作为参数。第一个回调函数是Promise对象的状态变为resolved时调用，第二个回调函数是Promise对象的状态变为rejected时调用。其中，第二个函数是可选的，不一定要提供。这两个函数都接受Promise对象传出的值作为参数。


```
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
});
```
## Promise.prototype.then() 

Promise 实例具有then方法，也就是说，then方法是定义在原型对象Promise.prototype上的。它的作用是为 Promise 实例添加状态改变时的回调函数。前面说过，then方法的第一个参数是resolved状态的回调函数，第二个参数（可选）是rejected状态的回调函数。


```
function pms1() {
                    return new Promise(function(resolve, reject) {
                        setTimeout(function() {
                            console.log('执行任务1');
                            resolve('执行任务1成功');
                        }, 2000);
                    });
                }

                function pms2() {
                    return new Promise(function(resolve, reject) {
                        setTimeout(function() {
                            console.log('执行任务2');
                            resolve('执行任务2成功');
                        }, 2000);
                    });
                }

                function pms3() {
                    return new Promise(function(resolve, reject) {
                        setTimeout(function() {
                            console.log('执行任务3');
                            resolve('执行任务3成功');
                        }, 2000);
                    });
                }
                pms1().then(function(data) {
                        console.log('第1个回调：' + data);
                        return pms2();
                    })
                    .then(function(data) {
                        console.log('第2个回调：' + data);
                        return pms3();
                    })
                    .then(function(data) {
                        console.log('第3个回调：' + data);
                        return '还没完！该结束了吧！'
                    }).then(function(data) {
                        console.log(data);
                    });
```
采用链式的then，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个Promise对象（即有异步操作），这时后一个回调函数，就会等待该Promise对象的状态发生变化，才会被调用。

上面栗子是我以前学习的时候参照写的，此乃我写demo的第一个栗子。

## Promise.prototype.catch()

*Promise.prototype.catch* 方法是 *.then(null, rejection)* 的别名，用于指定发生错误时的回调函数。



```
p.then((val) => console.log('fulfilled:', val))
  .catch((err) => console.log('rejected', err));

// 等同于
p.then((val) => console.log('fulfilled:', val))
  .then(null, (err) => console.log("rejected:", err));
```
Promise 在resolve语句后面，再抛出错误，不会被捕获，等于没有抛出。因为 Promise 的状态一旦改变，就永久保持该状态，不会再变了。

Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个catch语句捕获。


```
getJSON('/post/1.json').then(function(post) {
  return getJSON(post.commentURL);
}).then(function(comments) {
  // some code
}).catch(function(error) {
  // 处理前面三个Promise产生的错误
});
```
一般来说，不要在then方法里面定义Reject状态的回调函数（即then的第二个参数），而应总是使用catch方法。

## Promise.all()

Promise.all方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。

语法很简单：参数只有一个，可迭代对象，可以是数组，或者Symbol类型等。
```
 Promise.all([
      new Promise(function(resolve, reject) {
        resolve(1)
      }),
      new Promise(function(resolve, reject) {
        resolve(2)
      }),
      new Promise(function(resolve, reject) {
        resolve(3)
      })
    ]).then(arr => {
      console.log(arr) // [1, 2, 3]
    })
```

p的状态由p1、p2、p3决定，分成两种情况。
1. 只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。
2. 只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。

## Promise.race()

**Promise.race()：** 语法和all()一样，但是返回值有所不同，race根据传入的多个Promise实例，只要有一个实例resolve或者reject，就只返回该结果，其他实例不再执行。
 

```
Promise.race([
      new Promise(function(resolve, reject) {
        setTimeout(() => resolve(1), 1000)
      }),
      new Promise(function(resolve, reject) {
        setTimeout(() => resolve(2), 100)
      }),
      new Promise(function(resolve, reject) {
        setTimeout(() => resolve(3), 10)
      })
    ]).then(value => {
      console.log(value) // 3
    })
```
给每个resolve加了一个定时器，最终结果返回的是3，因为第三个Promise最快执行。

## Promise.resolev()

Promise.resolve方法就起到的作用将现有对象转为Promise对象。

如果参数是Promise实例，那么Promise.resolve将不做任何修改、原封不动地返回这个实例。


如果参数是一个不具有then方法的对象，则Promise.resolve方法返回一个新的Promise对象，状态为resolved。

# Promise.reject()
Promise.reject(reason)方法也会返回一个新的 Promise 实例，该实例的状态为rejected。


```
var p = Promise.reject('出错了');
// 等同于
var p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function (s) {
  console.log(s)
});
// 出错了
```
## 两个有用的附加方法

ES6的Promise API提供的方法不是很多，有些有用的方法可以自己部署。

### done()

Promise对象的回调链，不管以then方法或catch方法结尾，要是最后一个方法抛出错误，都有可能无法捕捉到（因为Promise内部的错误不会冒泡到全局）。因此，我们可以提供一个done方法，总是处于回调链的尾端，保证抛出任何可能出现的错误。


```
Promise.prototype.done = function (onFulfilled, onRejected) {
  this.then(onFulfilled, onRejected)
    .catch(function (reason) {
      // 抛出一个全局错误
      setTimeout(() => { throw reason }, 0);
    });
};
```
### finally()

finally方法用于指定不管Promise对象最后状态如何，都会执行的操作。它与done方法的最大区别，它接受一个普通的回调函数作为参数，该函数不管怎样都必须执行。


```
Promise.prototype.finally = function (callback) {
  let P = this.constructor;
  return this.then(
    value  => P.resolve(callback()).then(() => value),
    reason => P.resolve(callback()).then(() => { throw reason })
  );
};
```

## Promise和异步的联系
Promise本身不是异步的，只有他的then()或者catch()方法才是异步，也可以说Promise的返回值是异步的。通常Promise被使用在node，或者是前端的ajax请求、前端DOM渲染顺序等地方。
### async 函数
下一节总结。

## 总结

这一节大概需要掌握的就是Promise是什么、怎么用、怎么获取返回值？
一些概念性的东西多看看把。