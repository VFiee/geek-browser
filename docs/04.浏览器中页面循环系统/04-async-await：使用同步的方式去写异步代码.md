# async/await：使用同步的方式去写异步代码

## async/await 是如何实现的

async/await 使用 Generator 和 Promise 两种技术实现

### Generator & Coroutines

[Generator MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Generator)

```js
function* genDemo() {
  console.log("开始执行第一段")
  yield "generator 2"

  console.log("开始执行第二段")
  yield "generator 2"

  console.log("开始执行第三段")
  yield "generator 2"

  console.log("执行结束")
  return "generator 2"
}

console.log("main 0")
let gen = genDemo()
console.log(gen.next().value)
console.log("main 1")
console.log(gen.next().value)
console.log("main 2")
console.log(gen.next().value)
console.log("main 3")
console.log(gen.next().value)
console.log("main 4")
```

什么是协程?

协程是基于线程又比线程更轻量级的存在,由开发者写程序来维护并管理的轻量级线程

协程的特点

1. 线程的切换由操作系统负责调度，协程由用户自己进行调度，减少了上下文切换，提高了效率。
2. 线程的默认 Stack 大小是 1M，而协程更轻量，接近 1K。因此可以在相同的内存中开启更多的协程。
3. 由于在同一个线程上，因此可以避免竞争关系而使用锁。
4. 适用于被阻塞的，且需要大量并发的场景。但不适用于大量计算的多线程，遇到此种情况，更好的方式是使用线程去解决。
5. 在线程上同时只能执行一个协程

![协程执行流程图](../../images/04/coroutines-process.png)

疑问: 父协程有自己的调用栈，gen 协程时也有自己的调用栈，当 gen 协程通过 yield 把控制权交给父协程时，V8 是如何切换到父协程的调用栈？当父协程通过 gen.next 恢复 gen 协程时，又是如何切换 gen 协程的调用栈？

1. gen 协程和父协程是在主线程上交互执行的，并不是并发执行的，它们之间的切换是通过 yield 和 gen.next 来配合完成的。
2. 当在 gen 协程中调用了 yield 方法时，JavaScript 引擎会保存 gen 协程当前的调用栈信息，并恢复父协程的调用栈信息。同样，当在父协程中执行 gen.next 时，JavaScript 引擎会保存父协程的调用栈信息，并恢复 gen 协程的调用栈信息。

![调用栈切换](../../images/04/stack-switch.png)

### async

[async MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)

```js
async function foo() {
  console.log(1)
  let a = await 100
  console.log(a)
  console.log(2)
}
console.log(0)
foo()
console.log(3)
```

1. 执行 console.log(0)这个语句，打印出来 0
2. 执行 foo 函数，foo 函数有 async 标记，进入该函数时，JavaScript 引擎会保存当前的调用栈信息，然后执行 foo 函数中的 console.log(1)语句，并打印出 1。
3. 执行到 foo 函数中的 await 100 语句,创建一个 Promise 对象,` let promise_ = new Promise((resolve,reject){ resolve(100) })`,Promise 对象调用 resolve 函数,JavaScript 引擎将`resolve(10)添加到微任务列表`,JavaScript 引擎暂停 foo 的子协程执行,主线程控制权回到父协程 foo,同时将\_promise 对象返回,父协程随后便调用\_promise.then 监听 Promise 的状态
4. 执行 console.log(3),并打印出 3
5. 父协程将结束,检查点检查到有微任务待执行,并执行微任务,`resolve(100)`,触发子协程,并将 value 传递给 a,并打印 a
6. 执行结束

![示例图解](../../images/04/async-await.png)
