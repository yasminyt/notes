# Promise
+ 一个 `promise` 对象的语法结构如下：
  ```javascript
  let promise = new Promise(function(resolve, reject) {
    // ...
  })
  ```
  + 传递给 `new Promise` 的函数称为「执行程序」
  + 创建 `promise` 时，执行程序会自动运行
+ 生成的 `promise` 对象具有两个内部属性：
  + **`state`**：初始化为「pending」，然后更改为「fulfilled」或者「rejected」
  + **`result`**：任意值，初始为 `undefined`
+ 当执行程序完成相应的任务时，就会调用其参数中的一个函数：
  + **`resolve(value)`**：表示任务成功完成
    + 设置 `state` 为「fulfilled」
    + 设置 `result` 为 `value`
  + **`reject(value)`**：表示发生了错误
    + 设置 `state` 为「rejected」
    + 设置 `result` 为 `error`
  
  ![fig1](https://javascript.info/article/promise-basics/promise-resolve-reject.png)

+ 例如在以下例子中，创建了一个简单的 `promise`
  ```javascript
  let promise = new Promise(function(resolve, reject) {
    // 当创建了 promise 时，这个函数就会自动执行

    // 1s后，任务完成，并且结果为“done”
    setTimeout(() => resolve('done'), 1000)
  })
  ```
  + 通过上述代码可以知道：
  1. 执行程序会立即自动调用（通过 `new Promise`）
  2. 执行程序接受两个参数：`resolve` 和 `reject` —— 这两个函数是由 JS 引擎预定义的，所以不需要创建它们，只需要视情况进行调用即可
  + 那么上述代码在执行完成后，可以由以下结构图来表示：

  ![fig2](https://javascript.info/article/promise-basics/promise-resolve-1.png)

+ 那么也可以是 `reject` promise，并且抛出错误：
  ```javascript
  let promise = new Promise(function(resolve, reject) {
    setTimeout(() => reject(new Error('Whoops')), 1000)
  })
  ```
  + 对应的结构图如下：

  ![fig3](https://javascript.info/article/promise-basics/promise-reject-1.png)

+ 总的来说，执行程序需要完成相应的处理，然后调用 `resolve` 或 `reject` 来改变相应的 Promise 对象的状态
+ 「resolved」或者「rejected」的 Promise 称为「settled」，而不是最初的「pending」状态

### 只执行第一个 resolve/reject
+ 执行程序应该只调用一次 `resolve` 或 `reject`，那么 promise 的状态就已经被改变
+ 如果多次调用 `resolve` 或 `reject`，就会被忽略
  ```javascript
  let promise = new Promise(function(resolve, reject) {
    resolve('done')  

    reject(new Error('...'))    // 忽略
    setTimeout(() => resolve('...'))    // 忽略
  })
  ```
+ 另外，`resolve` / `reject` 只接受一个参数（或没有），其余的参数将会被忽略

### reject 使用 Error 对象
+ `reject` 的参数和 `resolve` 一样可以是任意类型
+ 但是推荐使用 `Error` 对象（或者继承自 `Error` 的对象）

## then, catch, finally
+ 一个 Promise 对象充当了执行函数与消费函数之间的链接，消费函数是指接受执行函数的结果或错误，并作出相应的操作
+ 消费函数通常有 `.then`、`.catch` 和 `.finally` 这几种方式

### then
+ `.then` 是最重要、最基础的
+ 基本形式如下：
  ```javascript
  promise.then(
    function(result) {
      // 处理成功的结果
    },
    function(error) {
      // 处理产生错误时
    }
  )
  ```
+ `.then` 的第一个参数是匿名函数，作用是：
  1. 当 promise 为「resolved」时执行
  2. 接受 result（作为其参数）
+ `.then` 的第二个参数也是匿名函数，作用是：
  1. 当 promise 为「rejected」时执行
  2. 接受 error
+ `.then` 的两个匿名函数参数根据 promise 的状态对应运行，即每次只会运行一个函数
  ```javascript
  // 例如当 promise 的状态变为 resolved 时，运行第一个函数
  let promise = new Promise(function(resolve, reject) {
    setTimeout(() => resolve('done'), 1000)
  })

  promise.then(
    result => alert(result)   // 1s 后弹框显示 done
    error => alert(error)     // 不执行
  )
  ```
+ 如果只关心执行成功的状态，那么可以只提供一个参数，则只有在 Promise 为「resolved」时才执行

### catch
+ 如果只关心执行异常时的状态，即 Promise 为「rejected」，那么可以使用 `.catch`
+ `.catch` 只关心异常的状态，直接捕获异常
+ `.catch(f)`：接受一个匿名函数进行处理，匿名函数的参数也为 `error`，即完整的形式为：`.catch(function(error) {...})`
  ```javascript
  let promise = new Promise((resolve, reject) => {
    setTimeout(() => reject(new Error('Whoops')), 1000)
  })

  // 将 alert 函数作为匿名函数
  promise.catch(alert)    // 1s 后弹框显示 Error: Whoops
  ```
+ `.catch(f)` 相当于 `.then(null, f)`，不过 `.catch` 更简洁
+ `.catch` 可以捕获 Promise 的执行函数中抛出的异常，就像 `try...catch` 类似，即在执行函数中不一定要调用 `reject(new Error(message))`，而是可以通过 `throw new Error(message)` 的方式（详见 promise 的错误处理中的「隐藏的 try...catch」内容）

### finally
+ `.finally(f)` 类似于 `.then(f, f)`，即不管 Promise 是 resolved 还是 rejected，都会执行 `.finally` 中的程序
+ `.finally` 可以用于执行一些清理程序，例如清除某些缓存等
+ 但是 `.finally(f)` 并不等同于 `.then(f, f)`，有以下几点不同：
  1. `.finally` 的匿名函数没有参数，即 `.finally(function() {...})`；因为 `.finally` 只关心 Promise 的执行函数是否已完成，即 Promise 的 state 是否已发生变化，但并不关心是 resolved 还是 rejected，所以并不处理结果或异常
  2. `.finally` 会将 结果或异常 传递给 `.then` 或 `.catch`
    + 例如在以下代码中，`.finally` 将 result 传递给 `.then`
      ```javascript
      new Promise((resolve, reject) => {
        setTimeout(() => resolve('result'), 1000)
      })
        .finally(() => alert('Promise ready'))
        .then(result => alert(result))
      ```
    + `.finally` 将 error 传递给 `.catch`
      ```javascript
      new Promise((resolve, reject) => {
        throw new Error('error')
      })
        .finally(() => alert('Promise ready'))
        .catch(error => alert(error))
      ```
  3. `.finally(f)` 比 `.then(f, f)` 更简洁：不需要赋值函数 `f`（不太明白🤔️）

## Promise & 异步
+ Promise 的执行函数是立即执行的
+ `.then`、`.catch` 和 `.finally` 都是异步的
+ 当执行函数完成时（除了延迟操作，如 `setTimeout`），即便立即改变了 Promise 的状态（resolved/rejected），都不会立即执行 `.then`、`.catch` 和 `.finally`，而是先执行 Promise 外的其它同步代码，只有这些同步代码执行完成，才会执行 `.then`、`.catch` 和 `.finally`
  ```javascript
  setTimeout(() => console.log('forth'), 1000)            // (1)
  new Promise((resolve, reject) => console.log('first'))  // (2)
    .then(result => console.log('third'))                 // (3)
  console.log('second')                                   // (4)
  setTimeout(() => console.log('fifth'), 1000)            // (5)

  // 输出顺序依次为：first、second、third、forth、fifth
  ```
  + (1) `setTimeout` 的延迟操作，所以该任务将放到了循环队列中
  + (2) Promise 的执行函数是立即执行的，所以就先打印出了“first”
  + (3) `.then` 是异步执行的，所以依然是被放到了循环队列中（但是与 `setTimeout` 这样的延迟操作的队列不是一样的，这里涉及到事件的先后处理顺序问题）
  + (4) 同步脚本，所以直接进行打印，输出了“second”
  + (5) 又是一个 `setTimeout` 的延迟操作，所以也被放到了循环队列中
  + 此时已经没有其它同步脚本，那么就会从循环队列中执行等待的任务：先执行 `.then` 中的任务，即 (3) 先执行，所以输出了“third”；然后执行 `setTimeout` 的任务队列，这里按照先后顺序，所以先执行 (1) 再执行 (5)，先后输出了“forth”、“fifth”

## Example：loadScript
+ 以脚本加载的为例，说明使用Promise比使用回调函数的优势
+ 回调函数：
  ```javascript
  function loadScript(src, callback) {
    let script = document.createElement('script')
    script.src = src
    
    script.onload = () => callback(null, script)
    script.onerror = () => callback(new Error(`Script load error for ${src}`))

    document.head.append(script)
  }
  ```
+ Promise：
  ```javascript
  function loadScript(src) {
    return new Promise((resolve, reject) => {
      let script = document.createElement('script')
      script.src = src
      
      script.onload = () => resolve(script)
      script.onerror = () => reject(new Error(`Script load error for ${src}`))

      document.head.append(script)
    })
  }

  let promise = loadScript("https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.11/lodash.js")

  promise.then(
    script => alert(`${script.src} is loaded`),
    error => alert(`Error: ${error.message}`)
  )
  // 再一次调用 .then
  promise.then(script => alert('one more handler to do something else'))  
  ```
+ 从上述例子中，可以看到使用 Promise 的一些优势：
  
  | Promises | Callbacks |
  | -------- | --------- |
  | Promise 可以以自然的顺序来编写逻辑代码。例如，先执行 `loadScript(script)`，然后在 `.then` 中根据结果进行相应的处理 | 当调用 `loadScript(script, callback)` 时，必须要有一个回调函数，也就是说在调用 `loadScript` 之前必须知道如何处理结果 |
  | 可以在一个 Promise 实例中多次调用 `.then`，只要有需要 | 只能有一个回调函数 |