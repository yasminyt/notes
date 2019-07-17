# Promise API

## Promise.resolve
+ 语法如下：
  ```javascript
  let promise = Promise.resolve(value)
  ```
  + 返回具有给定值的「resolved」promise
  + 相当于：
    ```javascript
    let promise = new Promise(resolve => resolve(value))
    ```
+ 该方法主要用于：已经知道 resolve 的最终结果，但是想要将其嵌套在一个 promise 中
+ 例如在以下例子中，`loadCached` 函数获取 `url` 并通过简单的缓存来记住结果，以便将来对同一 URL 的调用可以立即返回
  ```javascript
  function loadCached(url) {
    let cache = loadCached.cache || (loadCached.cache = new Map())

    if (cache.has(url))
      return Promise.resolve(url)   //（*）

    return fetch(url)
          .then(response => response.json())
          .then(text => {
            cache.set(url, text)
            return text
          })
  }
  ```
  + 那么不管是第一次加载 url 的内容还是以后，都可以通过 `loadCached(url).then(...)` 来处理，因为保证了返回的是一个 promise
  + 这也就是 `Promise.resolve()` 在行（*）中的用途：即确保接口是统一的，那么就总是可以在 `loadCached` 后使用 `.then`

## Promise.reject
+ 语法如下：
  ```javascript
  let promise = Promise.reject(error)
  ```
  + 创建一个有异常的「rejected」promise
  + 相当于：
    ```javascript
    let promise = new Promise((resolve, reject) => reject(error))
    ```
+ 该语法比较少用到

## Promise.all
+ 如果有多个并行的 promise 要执行，并且需要等待所有的任务都完成，那么就需要 `Promise.all` 来进行监听
+ 语法如下：
  ```javascript
  let promise = Promise.all([...promises..])
  ```
  + 参数通常是一个数组列表，其元素为 promise
  + 当所有的 promise 的执行程序都执行完毕时，其结果会构成一个新的数组作为 `Promise.all` 的最终结果
+ `Promise.all` 是最常用的一种方法
+ 如下代码所示，得到了 `[1, 2, 3]` 的结果：
  ```javascript
  Promise.all([
    new Promise(resolve => setTimeout(resolve(1), 3000)),   // 1
    new Promise(resolve => setTimeout(resolve(2), 2000)),   // 2
    new Promise(resolve => setTimeout(resolve(3), 1000))
  ]).then(alert)    // [1, 2, 3]
  ```
  + `Promise.all` 将会等待所有的 promise 执行完毕时才开始调用 `.then` 
  + 几个 promise 的输出结果是相当于其原本定义的顺序的，所以即便 `setTimeout` 中等待的时间短，但是由于代码的顺序，所以还是最后输出
+ 一个常用的技巧是，将任务数组数组通过 `map` 得到对应的 promise 数组，然后再将其作为 `Promise.all` 的参数
  + 例如有一个 url 数组，那么就可以通过 `map` 转换为对应的 `fetch` 形式，以获得所有的资源：
    ```javascript
    let urls = [
      'https://api.github.com/users/iliakan',
      'https://api.github.com/users/remy',
      'https://api.github.com/users/jeresig'
    ];

    let requests = urls.map(url => fetch(url))

    Promise.all(requests)
          .then(responses => responses.forEach(
            response => alert(`${response.url}: ${response.status}`)
          ))
    ```
+ 一个更常见的用法是，根据几个人的名字获取他们对应的信息：
  ```javascript
  let names = ['iliakan', 'remy', 'jeresig'];

  let requests = names.map(name => fetch(`https://api.github.com/users/${name}`));

  Promise.all(requests)
        .then(responses => {
          // 可以显示所有请求的状态码
          responses.forEach(response => alert(`${response.url}: ${response.status}`))

          return responses    // 继续转发该结果
        })
        .then(responses => Promise.all(responses.map(r => r.json())))
        .then(users => users.forEach(user => alert(user.name)))
  ```
+ `Promise.all` 中如果有任何一个 promise 为「rejected」，那么就会立即 reject，并且捕获该异常，即便其它的 promise 为「resolved」，其结果也会被忽略掉
  ```javascript
  Promise.all([
    new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
    new Promise((resolve, reject) => setTimeout(() => reject(new Error("Whoops!")), 2000)),
    new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
  ]).then(alert)    // 没有执行
  .catch(alert);    // Error: Whoops!
  ```
  + 上述代码的执行结果可以知道，即便成功执行了某些 promise，但是 `.then` 也不会被执行，而是由 `.catch` 捕获了错误
+ 当某个 promise 出错时，`Promise.all` 的处理方式并不友好，因为忽略掉了其它正常执行并获得正确结果的 promise
  + 一种处理方式是，给每个 promise 带上 `.catch`，由 promise 本身去捕获异常，这样 `Promise.all` 的 `.then` 就可以正常获得结果
  ```javascript
  Promise.all(
    urls.map(url => fetch(url).catch(err => err))
  )  
  ```
+ **`Promise.all` 的参数列表中可以接受非 promise 的元素**
  + 如下代码的结果也为 `[1, 2, 3]`
    ```javascript
    Promise.all([
      new Promise(resolve => setTimeout(resolve(1), 1000)),
      2,  // 被视为是 Promise.resolve(2)
      3   // Promise.resolve(3)
    ]).then(alert)    // [1, 2, 3]
    ```

## Promise.race
+ `Promise.race` 与 `Promise.all` 类似，也是接受多个 promise 构成的参数列表，但是并不是等待所有的 promise 都执行完毕，而是只等待最早完成的那个 promise 的结果，那么其它的 promise 的结果就会被忽略
  ```javascript
  Promise.race([...promises...])
  ```
+ 例如在以下代码中，其输出结果为 1:
  ```javascript
  Promise.race([
    new Promise((resolve, reject) => setTimeout(resolve(1), 1000)),
    new Promise((resolve, reject) => setTimeout(reject(new Error('Whoops')), 2000)),
    new Promise((resolve, reject) => setTimeout(resolve(3), 3000))
  ]).then(alert)    // 1
  ```
  + 第一个 Promise 最早完成，所以 `Promise.race` 获得该结果并输出