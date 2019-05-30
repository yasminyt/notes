# 柯里化 & 偏函数

## 偏函数
+ 当确定一个函数的一些参数时，返回的函数（更加特定）被成为**偏函数**
+ 对于函数的 `bind` 方法来说，不仅可以绑定 `this` 的引用，还可以绑定参数
+ `bind` 方法的完整表达式为：
  ```javascript
  let bound = func.bind(context, arg1, arg2, ...)
  ```
  + 与 `call` 类似，也接受一个参数列表
  + 不同的是 `bind` 返回的是一个 `func` 对应的一个特殊函数，并且传入的参数将会实例化 `func` 中对应位置的参数
    ```javascript
    function mul(a, b) {
      return a * b
    }

    let double = mul.bind(null, 2)  // 实例化了 mul 中的参数 a，并返回该函数给 double

    alert(double(3))  // 相当于执行了 mul(2, 3) = 6
    alert(double(4))  // 相当于执行了 mul(2, 4) = 8
    alert(double(5))  // 相当于执行了 mul(2, 5) = 10
    ```
+ 偏函数应用「partial function application」：通过修复现有参数来创建一个新的函数
  + 如上例所示即为偏函数的一个实际应用例子，创建了一个基数为 2 的新的函数，并在此基础上执行原有的函数
  + 由于 `bind` 需要 `context` 参数，而这里没涉及到具体的 `this` 指向，所以直接赋值为 `null` 即可
+ 为什么使用偏函数：
  + 可以创建一个具有可读名称的独立函数（如上例中，在 `mul` 函数的基础上，新创建了一个 `double` 函数，表示其中一个乘数为 2，也可以创建 `triple` 函数，表示其中一个乘数为 3，依此类推），那么在使用这些新的独立函数时，不需要每次都提供第一个参数，因为它是用 `bind` 修复的
  + 当具有非常通用的功能（如 `mul` 函数），但是为了方便，有的时候想要一个不太通用的变体时（如 `bind` 函数）
  + 再如该应用场景：有一个通用的函数 `send(from, to, text)`，然后在一个 `user` 对象内部，则可以创建一个局部方法：`sendTo(to, text)`，表示均由该对象发送
+ 当我们不想一遍又一遍重复相同的参数时，偏函数可以是很方便的一种实现

### 通过包装器「wrapper」创建偏函数
+ 除了使用 `bind` 方法，还可以自定义一个包装器来实现偏函数的创建
  ```javascript
  /**
   * 创建一个包装器
   * [argsBound] 接收需要先实例化的参数（相当于上例中，bind 中的参数）
   */
  function partial(func, ...argsBound) {
    return function(...args) {    // 接收剩余的参数
      return func.call(this, ...argsBound, ...args)
    }
  }

  let user = {
    name: 'John',
    say(time, phrase) {
      alert(`[${time}] ${this.name}: ${phrase}`)
    }
  }

  // 创建一个偏函数，并且修复了第一个参数
  user.sayNow = partial(user.say, '10:00')

  alert(user.sayNow('Hello'))   // [10:00] John: Hello
  ```
  + 从上述代码中可以看到，包装器「`partial`」的实现相当于 `bind` ：使用了 `argsBound` 来接受要修复的参数，即先实例化的参数，相当于 `bind` 中的参数列表
  + 具体过程如下：
    1. 执行 `partial(user.say, '10:00')`，则 `argsBound` 中获得实参 '10:00'，并返回一个函数，所以新方法 `user.sayNow` 中的内容为 `function(...args) {...}`
    2. 执行 `user.sayNow('Hello')`，则 `args` 获得实参 'Hello'，并执行 `func()`，即 `user.say()`，绑定 `this = user`，并将参数 `argsBound` 和 `args` 分别传递给 `user.say()`，即完成了在「10:00 John」这个条件下的短语表达
    3. 也可以创建 `user.sayLater = partial(user.say, '13:00')` 这样的新的偏函数

## 柯里化「Currying」
+ 柯里化是一项将一个调用形式为 `f(a, b, c)` 的函数转换为调用形式为 `f(a)(b)(c)` 的技术
  ```javascript
  function curry(func) {
    return function(a) {
      return function(b) {
        return func(a, b)
      }
    }
  }

  // 原函数
  function sum(a, b) {
    return a + b
  }

  let curriedSum = curry(sum)

  alert( curriedSum(1)(2) )   // 3
  ```
  + 如上所示，柯里化的实现主要是一系列的「封装」
  + `curry()` 的结果就是一层封装；执行 `curry(sum)` 后，`curriedSum` 为返回的函数 `function(a) {...}`
  + 执行 `curriedSum(1)`，创建函数 `function(a) {...}` 的词法环境，并且保存了参数 `a = 1`，并且又返回一层新的封装，即函数 `function(b) {...}`，并且其 `[[environment]]` 属性指向 `function(a) {...}` 的词法环境
  + 同理，执行 `curriedSum(1)(2)`，创建函数 `function(b) {...}` 的词法环境，并且保存了参数 `b = 2`；这个时候执行 `func()`，从词法环境中找到参数 `a` 和 `b` 的值，即执行 `sum(1, 2)`，返回结果
+ 在 js 中，通常保留函数「被正常调用」和在「参数数量不够时返回偏函数」这两个特性，即以下所说的 高级柯里化

### 高级柯里化
+ 高级柯里化同时允许原函数的正常调用和使用偏函数的调用
  + 即原函数经过柯里化后依然能够正常调用，而且也允许偏函数的调用
  + 如下代码所示：
    ```javascript
    function func(a, b, c) {
      return a + b + c
    }
    // 将函数柯里化
    func = curry(func)
  
    // 按照一般形式调用 func，依然正常运行
    func(1, 2, 3)   // 6
    
    // 用柯里化格式调用也正常运行
    func(1)(2)(3)   // 6
    ```
+ 高级柯里化的简单实现：
  ```javascript
  function curry(func) {
    return function curried(...args) {
      // 判断获取到的参数列表的长度与函数所需参数的长度
      if (args.length >= func.length)
        return func.apply(this, args)
      else {
        return function(...args2) {
          return curried.apply(this, args.concat(args2))
        }
      }
    }
  }

  function sum(a, b, c) {
    return a + b + c
  }

  let curriedSum = curry(sum)

  // 依然可以被正常调用
  curriedSum(1, 2, 3)   // 6

  // 得到 curried(1) 的偏函数，然后用另外两个参数调用它
  curriedSum(1)(2, 3)   // 6

  // 完全柯里化形式
  curriedSum(1)(2)(3)   // 6
  ```
  + 上述代码中，主要是使用了递归去获取完整的参数列表，当已经获得原函数所需的完整参数个数时，则停止递归，执行函数；否则继续返回一个偏函数，以继续接收参数
  + 对于完全柯里化的调用形式来说，`curriedSum(1)(2)(3)` 的执行过程如下：
  1. 第一次 `curriedSum(1)` 之后，返回的是一个偏函数 `function(...arg2) {...}`，并且在外层词法环境中，`args = [1]`
  2. 第二次 `curriedSum(1)(2)` 之后，返回的也还是一个偏函数 `function(...arg2) {...}`（后续即便还有很多个参数，也都是返回这个偏函数，相当于用 `args2` 接收继续传递的参数）
    > 这是因为在「1」的基础上，执行了 `function(...arg2)`，并且 `args2 = [2]`
    > 然后递归调用 `curried()`，并且设置参数 `args = [1, 2]`
    > 因为长度还没满足，所以执行 `else` 语句，则返回的还是 `function(...arg2) {...}`
  3. 执行第三次 `curriedSum(1)(2)(3)` 时，执行「2」返回的 `function(...arg2)` 函数，则 `args2 = [3]`；同样的，递归调用 `curried()`，并且设置参数 `args = [1, 2, 3]`；因为此时 `if` 条件满足，所以执行 `func()`，即 `sum()`，并传参为 `[1, 2, 3]`（因为使用的是 `apply` 作调用，所以可以传递数组，相当于 `sum.apply(null, [1, 2, 3]`）
+ 根据定义，上述例子中，应该是将 `sum(a, b, c)` 转换为 `sum(a)(b)(c)` 也可以正常执行，这里借助 js 的库实现会更方便些

## Example
[偏函数在登录中的应用](https://zh.javascript.info/currying-partials#pian-han-shu-zai-deng-lu-zhong-de-ying-yong)