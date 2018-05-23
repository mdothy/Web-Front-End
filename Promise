#### Promise
###### 在开发中经常遇到这样的问题，当我们向后台发送一个请求时，经常会需要用到Promise，我们都知道，promisee是用来处理异步问题的，那么，为什么会出现这样的应用场景？以及promise内部是怎样来处理这些异步问题的呢？

在了解异步编程之前，我们需要理解以下的一些概念。
###### 1.同步与异步
###### 2.阻塞与非组塞
###### 3.事件循环
###### 4.回调函数
###### 5.promise


##### 同步与异步
javascript的单线程机制决定了它一次只能执行一个任务，也就是说，程序是顺序执行的。但是，如果遇到耗时量较大的操作的时候，如果等到这个任务执行完毕才执行下一个任务，那么，就会发生阻塞。异步就是用来处理这类问题的，当遇到请求网络资源，I/O操作等需要花费大量时间的任务时，javascript会把这些问题交给它的回调函数来处理，等到处理完毕之后返回所得结果，但是后一个任务，是不会等待前一个任务结束后才开始执行。这里用一个定时器来说明：

```
function test(){
	console.log("first");
    setTimeout('console.log("second")',1000);
    setTimeout('console.log("third")',100);
    console.log("four");
}
```

```
输出结果：first  four  third  second
```
在这里，定时器就是一个异步任务，而对耗时操作不一样的任务而言，它的输出结果顺序也很值得探讨

javascript对任务的处理顺序由函数调用栈和消息队列决定的，二者之间由明确的优先级，进入函数调用栈中的任务被一一执行，等到栈空的时候，才会执行消息队列中的函数。函数调用栈使我想到了它的执行环境：**每个函数都有自己的执行环境。当执行流进入到一个函数时，函数的环境就会被推入到一个环境栈中，而在函数执行之后，栈将其环境弹出，把控制权返回给当前的执行环境**
在这里，函数调用栈和消息队列请参考这篇文章[https://segmentfault.com/a/1190000014874668](http://)

##### 阻塞与非组塞
异步就是将阻塞问题变成非阻塞问题，它将不能立即得到的结果放在未来的某个时间点上进行输出或者返回。


###### 事件循环
通过上面的例子，我们可以看到，javascript先执行同步任务，然后再执行异步任务。在javascritp中，同步任务被压入栈中，而异步任务被压入堆中，压入堆中的任务通过事件循环来进行处理。只有当栈中的事件被一一处理，栈空的情况下，才会去处理堆中的事件。
在这里，事件循环可以参考这篇文章：[http://www.ruanyifeng.com/blog/2014/10/event-loop.html](http://)

##### 回调函数
回调函数时分为同步回调跟异步回调，同步回调意味着函数时顺序执行的，异步回调与此相反。
######同步回调
```
function f1(callbak){
    console.log("satrt");
    callback("middle");
    console.log("end");
}

function callback(msg) {  
        console.log(msg);
}

f1(callback);

输出结果：satrt  middle  end
```
###### 异步回调

```
function f1(callbak){
    console.log("satrt");
    callback("middle");
    console.log("end");
}

function callback(msg) {  
        setTimeout(function(){console.log(msg);},100);
}

f1(callback);

输出结果：start end  middle
```

同步回调跟异步回调的区别在与是否由异步任务，而对于多个回调，会出现这样的情况
```
function finder(records, cb) {
    setTimeout(function () {
        records.push(3, 4);
        cb(records);
    }, 1000);
}
function processor(records, cb) {
    setTimeout(function () {
        records.push(5, 6);
        cb(records);
    }, 1000);
}

finder([1, 2], function (records) {
    processor(records, function(records) {
      console.log(records);
    });
});

输出结果: [1, 2, 3, 4, 5, 6]
```
从上面的例子可以看出，回调里面层层嵌套会使得阅读起来十分困难，即使将函数进行拆分也会有很多的耦合度。更细节的地方可以查看着这篇文章：[http://sporto.github.io/blog/2012/12/09/callbacks-listeners-promises/](http://)，而Promise的出现不仅简化了程序的写法，当然，它也有它的一些内部特性

##### Promise
首先来看看Promise的定义，Promise时异步编程的一种解决方案，它是一个容器，里面保存着在未来结束的某个事件，而从语法上来说，它是一个对象，从它可以获取异步操作的信息。

###### 语法上的定义：
```
new Promise( function(resolve, reject) {...} /* executor */  );

```
在这里，先说明promise的3个状态：
```
1.pending:初始状态
2.fulfilled:意味着操作成功
3.rejected:意味着操作失败
```
executor函数里面会执行一个异步操作，而resolve和reject作为参数传递给executor，但resolve和reject本身就是函数。而resolve或reject的执行，会使得promise的状态会发生变化，执行resolve,promise就变成fulfilled，执行reject，promise就会变成rejected,根据promise的状态变化，promise对象的then方法绑定的处理方法就会被调用，对相应的错误进行处理或直接处理异步，最后，将结果返回给promise对象

说到这里，让我们再来看看回调，回调是用来解决异步操作的，但它的写法会让代码可读性变得困难，而promise也是为了解决异步，但是，它是一个代理，把异步操作放在函数内部，通过判断异步操作的执行状态，再根据执行状态进行处理，我们再来看一个例子：
```
let myFirstPromise = new Promise(function(resolve, reject){
    //当异步代码执行成功时，我们才会调用resolve(...), 当异步代码失败时就会调用reject(...)
    //在本例中，我们使用setTimeout(...)来模拟异步代码，实际编码时可能是XHR请求或是HTML5的一些API方法.
    setTimeout(function(){
        resolve("成功!"); //代码正常执行！
    }, 250);
});

myFirstPromise.then(function(successMessage){
    //successMessage的值是上面调用resolve(...)方法传入的值.
    //successMessage参数不一定非要是字符串类型，这里只是举个例子
    console.log("Yay! " + successMessage);
});

输出结果：Yay! 成功!
```
其中的then方法是处理执行状态的方法，promise支持链式调用，这种写法使得程序更加清晰，当然，如果出现错误，我们可以这样写
```
myFirstPromise(function(successMessage){
    //成功处理
}).catch(function(errorMessage){
    //错误处理
})
```

关于promise，这篇文章更加详细[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise](http://)
