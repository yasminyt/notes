# Promises chaining
+ 当有一系列的异步任务需要一个接一个完成时，那么可以通过 Promise 的「链」来方便的实现
+ Promise 的链可以简单表示为如下代码所示：
  ```javascript
  new Promise((resolve, reject) => {
    setTimeout(() => resolve(1), 1000)
  }).then(function(result) {
    alert(result)   // 1
    return result * 2
  }).then(function(result) {
    alert(result)   // 2
    return result * 2
  }).then(function(result) {
    alert(result)   // 4
    return result * 2
  })
  ```
  + 上述代码通过 `.then` 程序链将 `result` 不断进行传递（传递过程中还可以更改其值）
  + 该过程可以表示为如下所示：

  ![fig1](https://javascript.info/article/promise-chaining/promise-then-chain.png)

+ 该链实现的原因在于：调用 `promise.then`，`return` 返回的是一个 promise，并且该 promise 已经「resolved」，所以可以在此基础上继续调用 `.then`，并且能获得其结果
+ 需要注意的是，如果仅仅是对一个 promise 调用很多次 `.then`，那么并不是一个「链」，如下所示：
  ```javascript
  let promise = new Promise(function(resolve, reject) {
    setTimeout(() => resolve(1), 1000)
  })

  promise.then(result => {
    alert(result)   // 1
    return result * 2
  })

  promise.then(result => {
    alert(result)   // 1
    return result * 2
  })

  promise.then(result => {
    alert(result)   // 1
    return result * 2
  })
  ```
  + 对于一个 promise 来说，当它的最终状态确定之后，就不会再被更改，所以对它本身每次调用 `.then` 都会得到一样的 result
  + 上述代码对应的结构图如下所示：

  ![fig2](https://javascript.info/article/promise-chaining/promise-then-many.png)

## 返回 promise
+ 如果在 `.then` 程序链中，返回的是一个新的 promise，那么下一个 `.then` 就会获取这个新的 promise 的结果
  ```javascript
  new Promise((resolve, reject) => {
    setTimeout(() => resolve(1), 1000)
  }).then(result => {
    alert(result)   // 1
    return new Promise((resolve, reject) => {       // (*)
      setTimeout(() => resolve(result * 2), 1000)
    })
  }).then(result => {   // (**)
    alert(result)   // 2
    return new Promise((resolve, reject) => {
      setTimeout(() => resolve(result * 2), 1000)
    })
  }).then(result => {
    alert(result)   // 4
  })
  ```
  + 第一个 `.then` 返回的是一个新的 promise，见（*）行
  + 那么第二个 `.then` 获取的是（*）行的 promise 的结果，所以（**）的 `result` 的结果为 2
+ 返回一个新的 promise 可以建立异步操作链

### some examples: [loadScript](https://javascript.info/promise-chaining#example-loadscript) & [fetch](https://javascript.info/promise-chaining#bigger-example-fetch)