# 装饰&转发 call/apply

## 透明缓存
+ 假设说有某个CPU密集型的任务，但是它的结果是稳定的，即对于某个相同的参数，得到的结果都是一致的，那么对于这个任务来说，每次都要被调用，这显然是不合理的
+ 针对上述的问题，则需要在代码中创建一个缓存，以记住每次运行得到的结果，对于已经在缓存中的结果，则直接返回，不需要再重新进行计算，从而提高效率
+ 不是在原执行任务函数中创建这样的缓存，而是创建一个「装饰器」，可以实现更灵活的应用
```javascript
function slow(x) {
  // ...可能这里执行了一个繁重的CPU密集型任务
  return result   // 返回执行结果
}
// 创建缓存装饰器
function cachingDecorator(func) {
  let cache = new Map()
  return function(x) {
    if (cache.has(x)) {
      return cache.get(x) // 直接返回结果，不用再计算
    }
    let result = func(x)  // 不存在，调用函数进行计算（*）
    cache.set(x, result)  // 创建缓存
    return result
  }
}
slow = cachingDecorator(slow)   // （**）

alert(slow(1))  // 将参数1对应的计算结果写入缓存中
alert(`Again: ${slow(1)}`)  // 直接从缓存中读取结果

alert(slow(2))  // 将参数2对应的计算结果写入缓存中
alert(`Again: ${slow(2)}`)  // 直接从缓存中读取结果
```
+ 在上述代码中使用了闭包去解决该问题，需要特别注意（*）和（**）位置的代码
  + （**）这一行改变了 `slow` 函数的定义，通过在控制台中进行查看，可以证实此时 `slow` 的函数体为 `cachingDecorator()` 返回的嵌套函数
  + 而由于调用时 `cachingDecorator()`，将 `slow` 作为参数进行传递，所以在（*）这一行中，`func` 始终指向的是对初始 `slow` 的引用，通过在控制台进行查看，可以证实 `func` 的函数体始终为初始 `slow`的函数体（在上例中为 `return result`）
+ 在上述代码中，`cachingDecorator(func)` 的执行结果相当于一个包装器：返回的嵌套函数 `function(x)` 在其内部缓存逻辑中包装了对 `func(x)` 的调用，如下图所示：
  ![fig1](https://javascript.info/article/call-apply-decorators/decorator-makecaching-wrapper.png)
+ 总的来说，重新创建一个单独的 `cachingDecorator()` 而不是改变原有的任务逻辑代码，有以下几个好处：
  + `cachingDecorator(func)` 可重用，适应性更强，因为不需要考虑 `func` 具体是什么，做什么用，只需要将它执行，获得结果即可
  + 将新添加的缓存逻辑与原有代码分离（即没有在初始的 `slow()` 中添加任何新的代码，没有破坏其代码逻辑），没有增加原有代码自身的复杂性
  + 如果需要的话，还可以灵活增加更多的装饰器

## 使用 “func.call“ 改变上下文
+ 在上述例子中，对于对象中的方法有时并不奏效。例如，在下例中，`worker.slow()` 在使用了装饰器之后会停止工作
  ```javascript
  let worker = {
    someMethod() {
      return 1
    },
    slow(x) {
      // ... CPU密集型任务
      return x * this.somemethod()  // (*)
    }
  }

  // 先前所创建的装饰器
  function cachingDecoration(func) {
    let cache = new Map()
    return function(x) {
      if (cache.has(x))
        return cache.get(x)
      let result = func(x)  // （**）
      cache.set(x, result)
      return result
    }
  }
  alert(worker.slow(1)) // 初始的 slow 是正常运行的（this = worker）

  worker.slow = cachingDecoration(worker.slow)
  alert(worker.slow(2))  // Error: Cannot read property 'someMethod' of undefined (this = undefined)
  ```
  + 这是因为在（*）这一行有 `this` 需要被正确赋值，但是在（**）这一行中，`func()` 并没有获得 `slow` 的执行上下文，所以 `this` 没办法正确饮用对应的对象，即 `this = undefined`，相当于如下代码
    ```javascript
    let func = worker.slow
    func(2)
    ```
+ `func.call(context[, arg1, arg2, ...])`：该内置方法可以显示地给函数的 this 设置上下文
  + 运行 `func`，提供第一个参数作为 `this` 的指向，余下的作为 `func` 的参数
  ```javascript
  func(1, 2, 3)
  func.call(obj, 1, 2, 3)
  ```
  + 在上述代码中，都是调用 `func`，并且传参为 1、2、3；唯一的不同是，`func.call` 还将 `func` 内的 `this` 指向了 `obj`，即 `this = obj`
  + 例如，在以下代码中，通过将不同的对象作为 `context`，从而函数中的 `this` 有不同的值
    ```javascript
    function sayHi() {
      alert(this.name)
    }
    let user = { name: 'John' },
        admin = { name: 'Admin' }

    sayHi.call(user)  // 'John'   (this = user)
    sayHi.call(admin) // 'Admin'  (this = admin)
    ```
+ 所以将本节开篇的例子使用 `func.call` 对 `this` 进行正确引用
  ```javascript
  let worker = {
    someMethod() {
      return 1
    },
    slow(x) {
      // ... CPU密集型任务
      return x * this.somemethod()  
    }
  }

  // 先前所创建的装饰器
  function cachingDecoration(func) {
    let cache = new Map()
    return function(x) {
      if (cache.has(x))
        return cache.get(x)
      let result = func.call(this, x)
      cache.set(x, result)
      return result
    }
  }

  worker.slow = cachingDecoration(worker.slow)  // (*)
  alert(worker.slow(2))  // 2
  ```
  + 装饰后的 `worker.slow` 现在是包装函数 `function(x) {...}` （*）
  + 所以当执行 `worker.slow(2)` 时，这个包装函数获得 2 作为参数，并且 `this = worker`（即指向了 `.slow()` 之前的对象，即 `worker`）
  + 在包装函数中，假设结果还没有被缓存，则调用初始的 `worker.slow()` 函数进行运算

## 使用 “func.apply“ 传递多参数（类数组对象）
+ `func.apply(context, args)`：该方法可以将要传递给函数的形参所构成的类数组对象（包括数组）作为参数「`args`」进行传递
  + 运行 `func`，并设置 `this = context`，且使用类数组的对象 `args` 作为参数列表
    ```javascript
    function say(time, phrase) {
      alert(`[${time}] ${this.name}: ${phrase}`)
    }

    let user = { name: 'John' },
        messageData = ['10:00', 'Hello']

    say.apply(user, messageData)  // [10:00] John: Hello  (this = user)
    ```
+ `call` 和 `apply` 唯一的语法区别在于 `call` 接受一个参数列表，而 `apply` 则接受带有一个类似数组的对象（包括数组）
  + 但是通过扩展运算符 `...` 与 `call` 一起使用，就可以实现与 `apply` 几乎相同的功能
    ```javascript
    let args = [1, 2, 3]

    func.call(context, ...args)
    func.apply(context, args)
    ```
  + 虽说二者看上去在使用上已经并无太大差别，但是在这样的情况下（参数列表 即可迭代又像数组一样），那么使用 `apply` 可能会更快一些，因为它只是一个操作，大多数 JS 引擎对其的内部优化比一对「call + spread」更好
+ 对 `call` 中的缓存实例应用 `apply` 以使得其更健壮
  ```javascript
  let worker = {
    slow(min, max) {
      return min + max
    }
  }

  function cachingDecoration(func, hash) {
    let cache = new Map()
    return function() {
      // 多参数的情况，那么需要生成一个 key，所以需要对其进行简单的哈希处理
      let key = hash(arguments)   
      if (cache.has(key)) 
        return cache.get(key)

      let result = func.apply(this, arguments)  // 这里与调用 call 没有太大区别，唯一不同的是当参数不定时，可以直接使用 arguments 关键词作为参数进行传递
      cache.set(key, result)
      return result
    }
  }

  function hash(args) {
    return arg[0] + ',' + arg[1]   // 本例中已经知道是两个参数，在实际当中应该是更普适的代码
  }

  worker.slow = cachingDecoration(worker.slow, hash)

  alert(worker.slow(3, 5))
  ```

### 方法借用「method borrowing」
+ 在上例所提到的简单的哈希函数的实现中，对于多参数来说是不适用的，那么要实现像这样的简单将几个值进行连接，对于数组来说，最自然的是直接使用 `arr.join` 方法
+ 但是对于非数组来说（如上例中，`arguments` 只是个类数组的对象），却不能直接用 `arr.join` 方法，那么可以通过「方法借用」的方式来实现，例如：
  ```javascript
  function hash() {
    return [].join.call(arguments)
  }

  hash(1, 2)    // "1, 2"
  ```
  + 在上述代码中，从一个常规的数组中借用了 `join` 方法，即 `[].join`，并且使用 `[].join.call` 在 `arguments` 的上下文中运行它
  + 简单来说就是借用数组的 `join` 方法，然后将 `arguments` 中的每个元素按照`join` 方法的规则合并成一个字符串
+ 将数组的方法借用在一些类数组的对象上（如 `arguments`）是很常用的一种方式
+ （还是比较🤔，过后再补充）

## 未完待续……