# promise 中的错误处理
+ 当 promise 中产生了错误，或是 promise 的状态变为「rejected」时，则一般通过 `.catch` 来处理这样的情况
+ 虽然 `.then()` 也可以处理「rejected」，但是在 promise 链中通过 `.catch` 处理会使程序逻辑更加清晰
+ 当 promise 链中发生异常或者「rejected」时，会由最近的一个 `.catch` 来捕获并处理

## 隐藏的 try...catch
+ 在 promise 的执行过程中，有一个隐藏的 `try...catch` 环绕着代码，如果发生异常时，则会被捕获，并被视为「rejected」
  + 例如在以下代码中，抛出了一个异常：
    ```javascript
    new Promise((resolve, reject) => {
      throw new Error('Whoops!')
    }).catch(alert)     // Whoops!
    ```
  + 则等同于执行了 `reject()`：
    ```javascript
    new Promise((resolve, reject) => {
      reject(new Error('Whoops!'))
    }).catch(alert)     // Whoops!
    ```
+ `try...catch` 并不只是在 执行函数中，也会发生在 `.then` 中，如果 `.then` 中产生了异常，也就是意味着这个新的 promise 变成了「rejected」，那么就会由最近的一个 `.catch` 捕获
  ```javascript
  new Promise((resolve, reject) => {
    resolve('ok')
  }).then(result => {
    throw new Error('Whoops!')
  }).catch(alert)   // Whoops!
  ```
+ 产生的异常并不局限于 `throw` 抛出来的，而是所有的可能的错误，例如语法错误
  ```javascript
  new Promise((resolve, reject) => {
    resolve('ok')
  }).then(result => {
    blabla()   // no such function
  }).catch(alert)   // ReferenceError: blabla is not defined
  ```

## Rethowing
+ 类似于 `try...catch` 的语法，在 promise 中，`.catch` 也可以将未处理的错误再继续抛出
+ 如果 `.catch` 继续抛出错误，那么将由下一个 `.catch` 继续进行捕获与处理
  ```javascript
  new Promise((resolve, reject) => {
    throw new Error('Whoops')
  }).catch(err => {
    if (err instanceof URIError) {
      // handle it
    } else {
      alert("Can't handle such error")

      throw err
    }
  }).then(() => {
    // 当再次抛出异常时，这里的代码永远不会被执行
  }).catch(err => {
    alert(`The unknown error has occurred: ${error}`)
  })
  ```
  + 在以上代码中，在第一个 `.catch` 中如果发生了不能处理的异常时，则会再次将该异常抛出
  + 若是第一个 `.catch` 再一次抛出了异常，则紧接着的 `.then` 将不会被执行，而是由第二个 `.catch` 捕获再次抛出的异常，并进行处理
+ 如果 `.catch` 已经将错误进行了处理，并且正常结束该执行流，那么此时相当于 promise 为「resolved」，所以后面接的 `.then` 将会被执行
  ```javascript
  new Promise((resolve, reject) => {
    throw new Error('Whoops')
  }).catch(err => {
    alert('The error is handled, continue normally')    // 没有再产生新的异常，catch 中的执行流正常结束
  }).then(() => alert('Next successful handler runs'))  // 继续执行 then 中的内容
  ```

### [Fetch error handling example](https://javascript.info/promise-error-handling#fetch-error-handling-example)

