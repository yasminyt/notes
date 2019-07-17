# Microtasks & 事件循环
+ promise 的处理程序 `.then`、`.catch`、`.finally` 总是异步的，尽管 promise 的状态是立即「resolved」或者「rejected」，`.then`、`.catch`、`.finally` **后**的代码也总是会先执行
  ```javascript
  let promise = Promise.resolve()

  promise.then(() => alert('promise done'))   //（*）

  alert('code finished')    //（**）
  ```
  + 在上述代码中，会先显示 “code finished”，然后才显示 “promise done”

## Microtasks
+ V8 引擎为异步任务的管理指定了一个内部队列 PromiseJobs，通常被称为「微任务队列，Microtasks」
+ 该队列具有以下特点：
  + 队列符合先进先出的规则：首先入队的任务先被运行
  + 只有在没有任何其它同步任务运行时才会启动队列中的任务
+ 当 promise 的执行程序完成时，promise 的处理程序 `.then`、`.catch`、`.finally` 都会进入到 microtasks 队列中，只有在当前脚本中没有其它同步脚本运行时，才会从该队列中逐个执行
+ 所以在开始的示例代码中，先执行了（**）行，然后再执行（*）行，如果想要按照预期来输出，那么需要都放到队列中才行：
  ```javascript
  Promise.resolve()
        .then(() => alert('promise done'))
        .then(() => alert('code finished'))
  ```
  + 两个 `.then` 都会被放入到 microtasks 队列中，按照先进先出原则，则会按顺序执行这两个 `.then`

## 事件循环
+ 浏览器内的 JavaScript 执行流以及 Node.js，都是基于「事件循环，event loop」
+ 「事件循环」是引擎休眠并等待事件的过程；当有事件发生 —— 处理它们 —— 再次休眠
+ 事件可能来自外部源，如用户操作，也可能来自内部任务的结束信号
+ 一些事件：
  + `mousemove`：用户移动鼠标
  + `setTimeout`：处理程序被调用
  + 外部的 `<script src='...'>` 被加载，准备被执行
  + 网络操作，如：`fetch` 完成
  + 等等……
+ 事件发生 —— 引擎处理它们 —— 等待更多的事情发生（当休眠时，对 CPU 的消耗几乎为 0），该过程如下图所示：

  ![fig1](https://javascript.info/article/microtask-queue/eventLoop.png)

+ 对于一般的事件来说，V8 引擎也提供了「macrotask 队列」来管理这些事件
  + macrotask 队列同样遵循了先进先出的规则，即先入队列的事件会先被处理
  + 如上图所示，会优先处理 `fetch` 事件，当其执行完时，再从 macrotask 队列中继续处理下一个事件，即 `mousemove` 事件，依次类推
  + 那么在引擎处理事件的过程中，如果有新的事件发生，那么就要进入到 macrotask 队列中等待
+ **microtask 队列相比于 macrotask 队列有更高的优先级**
  + 也就是说，引擎会优先处理 microtask 中的事件，然后才处理 macrotask 中的事件，所以 promise 的处理程序总是会优先被处理
  ```javascript
  setTimeout(() => alert('timeout'))

  Promise.resolve().then(() => alert('promise'))

  alert('code')
  ```
  + 对与上述代码来说，其先后输出的内容为：code、promise、timeout
    + `code` 先被输出，因为这是常规的同步调用
    + `promise` 其次输出，因为 `.then` 被放入到了 microtask 队列中，在所有的同步代码执行完时，随即被调用
    + `timeout` 最后输出，因为被放入到了 macrotask 队列中，最后才被执行

## 未处理的 reject
+ 「未处理 reject」是指在 microtask 队列结束时未处理的 promise 错误
+ 也就是说没有使用 `.catch` 去捕获错误，那么就会被引擎的 `unhandledrejection` 监听事件所捕获，而且也会在控制台输出该异常信息
  ```javascript
  Promise.reject(new Error('Promise failed'))

  window.addEventListener('unhandledrejection', event => {
    alert(event.reason)   // Promise failed
  })    // 也会在控制台显示异常信息
  ```
  + 对于该情况，只要加上 `.catch` 去捕获就可以了
    ```javascript
    Promise.reject(new Error('Promise failed'))
           .catch(err => alert('caught'))

    // 不会再被捕获到
    window.addEventListener('unhandledrejection', event => {
      alert(event.reason)   
    })    // 控制台也不会再显示异常信息
    ```
+ 如果将 `.catch` 放到了 `setTimeout` 中，那么还是会被 `unhandledrejection` 捕获到
  ```javascript
  let promise = Promise.reject(new Error('promise failed'))
  setTimeout(() => promise.catch(err => alert('caught')))

  window.addEventListener('unhandledrejection', event => {
    alert(event.reason)   // Promise failed
  })    
  ```
  + 由于 `.catch` 被放到了 `setTimeout` 中，而且 promise 已经变成「rejected」，引擎就会认为此时 microtask 队列已经是空的，所以 `unhandledrejection` 就会执行，即便后面 `.catch` 依然会被执行

## Summary
+ 总的来说，js 的事件处理先后顺序为：常规代码 —— promise 处理程序 —— 然后其它，如事件等