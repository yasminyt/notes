# Async & Await

## Async 函数
+ **`async`** 关键字可以放在一个函数生命之前，表示该函数返回的是一个 promise
  ```javascript
  async function f() {
    return 1
  }
  ```
  + 在上述代码中，尽管函数仍然只是简单的返回了一个结果，并且其函数体也与平时无异，但是由于 `async` 关键字的使用，该函数实际上会返回的是一个 promise
  + JavaScript 自动将 `return` 返回的内容作为 promise 变成「resolved」状态的结果，即相当于调用了 `resolve(1)`，上述代码可以更显示的写为如下所示：
  ```javascript
  async function f() {
    return Promise.resolve(1)
  }

  f().then(alert)   // 1
  ``` 

## Await
+ **`await`** 关键字只能用在 `async` 函数内，其语法如下：
  ```javascript
  // works only inside async functions
  let value = await promise
  ```
+ `await` 的作用在于：让 JavaScript 等待，直到 Promise 的状态改变，并且返回它的结果，也就是说 `await` 会阻塞当前上下文的脚本的执行，直到 promise 的状态改变
  ```javascript
  async function f() {
    let promise = new Promise((resolve, reject) => {
      setTimeout(() => resolve('done'), 1000)
    });

    let result = await promise

    console.log(result)
    console.log('test1')
  }

  f()
  console.log('test2')
  ```
  + 上述代码的执行结果为：test2、done、test1
  + 执行 `f()` 时，由于生成了一个 promise，并且其执行函数为 `setTimeout`，所以放到了 macrotask 队列中，又因为使用了 `await` 关键字，所以在 `f()` 函数的上下文环境中下面两行代码都被阻塞
  + 那么继续执行 `f()` 函数外的同步脚本，所以先输出了 `test2`
  + 此时没有同步脚本了，1s 后执行 `resolve`，所以 `result` 获得了结果为 `done`，则继续往下执行，依次输出 `done` 和 `test1`
+ `await` 关键字会阻塞当前上下文环境中脚本的执行，但是并不会占用 CPU，CPU仍然可以做别的事情，例如其它上下文环境中脚本的执行，如上例所示输出 `test2`；所以使用 `await` 可以保证先获得结果，然后再进行相应的处理
  ```javascript
  async function f() {
    let promise = new Promise((resolve, reject) => {
      setTimeout(() => resolve('done'), 1000)
    })

    promise.then(console.log)
    console.log('test1')
  }

  f()
  console.log('test2')
  ```
  + 上述代码依次输出的结果为：test1、test2、done
  + 通过比较这两段代码，可以明显看出了使用 `await` 的区别
+ `await` 声明的代码会被立即执行，但是由于 `await` 的阻塞性，所以该语句后的代码会被放入到 microtask 中，所以会先执行其它的同步代码
  ```javascript
  async function f() {
    console.log('test1');
    await f1()
    console.log('test2')    //（*）
  }
  function f1() {
    console.log('test3')
  }
  f();
  console.log('test4')
  ```
  + 上述代码的运行结果为：test1、test3、test4、test2
  + 由结果可以验证（*）行的代码被放入到了 microtask 队列中，即便 `f1()` 是一个立即执行的函数
+ `await` 不能用在普通的函数中，否则会产生语法错误
  ```javascript
  function f() {
    let promise = Promise.resolve(1)
    let result = await promise    // Syntax error
  }
  ```
  + `await` 只能用在 `async` 函数中
+ `await` 不能被用在顶级代码中，也就是说对于一些具有异步操作的函数，也不能直接使用 `await`
  ```javascript
  // Syntax error in top-level code
  let response = await fetch('/article/promise-chaining/user.json')
  let user = await response.json()
  ```
  + 如果想要将 `await` 作上述使用，可以将其包裹在一个 `async` 函数中
    ```javascript
    (async () => {
      let response = await fetch('/article/promise-chaining/user.json')
      let user = await response.json()
      // ...
    })
    ```
+ 即便在 `async` 中使用了 `await` 关键字，但是 `async` 函数返回的仍然是一个 promise
+ 也可以在 class 中添加 `async` 方法
  ```javascript
  class Waiter {
    async wait() {
      return await Promise.resolve(1)
    }
  }

  new Waiter().wait().then(alert)   // 1
  ```

## 异常处理
+ `await` 也接受 promise 的「rejected」状态，以及抛出其异常信息
  ```javascript
  async function f() {
    await Promise.reject(new Error('Whoops'))
  }
  ``` 
  + 等同于：
  ```javascript
  async function f() {
    throw new Error('Whoops')
  }
  ```
+ 同样是通过 `try...catch` 来捕获异常
  ```javascript
  async function f() {
    try {
      let response = await fetch('http://no-such-url')
    } catch(err) {
      alert(err)    // TypeError: failed to fetch
    }
  }

  f()
  ```
+ 如果在 `async` 中没有使用 `try...catch` 来捕获异常，那么也可以在函数外使用 `catch` 来捕获
  ```javascript
  async function f() {
    let response = await fetch('http://no-such-url')
  }

  // f() 已经变成一个 rejected promise
  f().catch(alert)    // TypeError: failed to fetch
  ```