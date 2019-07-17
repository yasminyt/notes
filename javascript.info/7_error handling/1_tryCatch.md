# 错误处理，try...catch

## try...catch 语法
+ `try...catch` 的语法如下：
  ```javascript
  try {
    // code...
  } catch (err) {
    // error handling
  }
  ```
  + 对应的处理过程如下：
  1. 首先，执行 `try {...}` 中的代码块
  2. 如果没有任何错误发生，那么 `catch (err)` 将会忽略：执行到达 `try` 结束，然后跳过 `catch`
  3. 如果产生了错误，`try` 就会立即停止，并且执行流转向开始执行 `catch (err)`。`err` 参数（可以是任意命名）是一个错误对象，包含了错误的详细内容
  + 对应的流程图如下所示：    
    ![fig1](https://javascript.info/article/try-catch/try-catch-flow.png)

+ 所以当 `try {...}` 中发生错误时并不会杀死脚本，而是可以在 `catch (err)` 中处理它

### try...catch 仅适用于运行时错误
+ 要使 `try...catch` 工作，必须是在脚本运行过程中，也就是说应该是在有效的 js 代码中
+ 如果代码具有语法错误，实际上浏览器就已经抛出错误，而不会正常执行 `try...catch`
  + 因为浏览器会先解析代码然后再运行，对于语法错误的代码，在解析过程就会抛出错误（“解析时错误”，并没有执行代码
+ 因此，`try...catch` 只能处理有效代码中出现的错误，这种错误称为是“运行时错误”，有时也称为是“异常”

### try...catch 同步工作
+ 如果在“schedule“的代码中发生异常，例如在 `setTimeout` 中，那么 `try...catch` 将无法捕获它
+ 例如在以下代码中，即便 `setTimeout` 中发生了错误，`try...catch` 也不能捕获到
  ```javascript
  try {
    setTimeout(() => {
      somethingWrong
    }, 1000)
  } catch(err) {
    alert("won't work")
  }
  ``` 
  + 对于 `setTimeout`/`setInterval` 等稍后执行的代码，js 引擎会先执行完所有的同步代码，然后再在消息队列中查找等待执行的函数
  + 所以对于上述代码，`setTimeout` 中的函数被放到了消息队列中稍后执行，那么 `try {...}` 中的内容就已经执行完，由于此时没有异常，所以忽略掉了 `catch(err)`，此时再执行 `setTimeout` 中的函数时，就算有异常也不会被捕获到
+ 要解决这样的问题，就是将 `try...catch` 放到 `setTimeout` 中，这样在执行 `setTimeout` 中的函数时，就会捕获相应的异常
  ```javascript
  setTimeout(function() {
    try {
      somethingWrong
    } catch(err) {
      alert('error is caught here!')
    }
  }, 1000)
  ```

## 错误对象
+ 当发生错误时，js 会生成一个包含了错误的详细信息的对象，然后将该对象作为参数传递给 `catch`
+ `catch(err)` 中的 `err` 对象主要包括了两个属性：
  + **`name`**：错误的名称，例如对于一个未定义的变量，则为 "ReferenceError"
  + **`message`**：有关错误详细信息的文本消息

## 可选的 catch 绑定
+ 当不需要理会具体错误信息时，可以忽略 `err` 参数，即 
  ```javascript
  try {
    // code
  } catch {
    // 忽略了 err 对象，直接对异常进行统一处理
  }
  ```
+ 这是新的语法特性，所以兼容性不是很好

## 自定义抛出异常
+ `catch` 默认捕获的是系统异常，而对于一些不算异常，但是希望能够被 `catch` 捕获的内容，则可以通过 `throw` 关键字来抛出异常
  ```javascript
  let user = {name: 'John'}
  try {
    alert(user.age)     // undefined
  } catch(err) {
    alert("doesn't execute")
  }
  ```
  + 例如在上述代码中，访问了一个未定义的属性，这对于 js 引擎来说并不是异常，但是对于开发者来说如果有这样的情况出现，那么希望能够即时获悉

### throw 操作符
+ `throw` 的形式为：
  ```javascript
  throw <error object>
  ```
  + 可以使用任何类型的数据作为一个 error object，如一个数字、一个字符、字符串等，但是最好还是使用对象，包含了错误的 name 和 message 等这些信息，像内置错误对象那样
+ js 有许多内置的构造函数用于创建标准的错误对象，如 `Error`、`SyntaxError`、`ReferenceError`、`TypeError` 等，所以可以使用它们来创建错误对象，形式如下：
  ```javascript
  let error = new Error(message)
  let error = new SyntaxError(message)
  let error = new ReferenceError(message)
  // ...
  ```
+ 对于这些内置的错误的构造函数，所生成的错误对象的 `name` 即为构造函数的 `name`，而错误的 `message` 即为所传递的参数，自定义的错误信息
  ```javascript
  let error = new Error('Things happen o_O')

  alert(error.name)   // Error
  alert(error.message)    // Things happen o_O
  ```
+ 那么使用自定义错误来修改代码：
  ```javascript
  let user = {name: 'John'}
  try {
    if (!user.age)
      throw new SyntaxError('no such property')  // (*)
    alert(user.age)     
  } catch(err) {
    alert("Object Error: " + err.message)   // Object Error: no such property
  }
  ```
  + 当 `user.age` 为 `undefined` 时，则会执行（*），`throw` 运算符使用给定的消息生成 `SyntaxError`，就像 js 自动生成错误那样
  + 那么就会停止执行 `try` 中的内容，由 `catch(err)` 捕获信息后，执行相应的错误应对操作

## Rethrowing
+ 在上述例子中，针对特定的某种错误作了自定义的异常抛出处理，但是如果在同一个地方有多种类型的错误，那么这样的处理显然是不合理的
+ 所以需要通过 `err.name` 去对各种不同的异常进行分别处理（甚至可以利用自定义的 `message` 来作区分）
+ 但是在做了所有可以做的处理之后，还是存在别的错误，那么应该再 `throw`
+ 应该遵循这样的规则：「`catch` 应该只处理它知道的错误并“重新抛出”所有其他错误」，具体来说就是：
  1. `catch` 获得所有的错误
  2. 在 `catch(err) {...}` 中，分析错误对象 `err`
  3. 如果不知道怎么处理时，就应该再 `throw err`
+ 例如在下例中，对未知错误的处理：
  ```javascript
  function readData() {
    let user = {name: 'john'}

    try {
      // ...
      blabla()   // Error!  
    } catch(err) {
      // ...
      if (err.name !== 'SyntaxError') {
        throw err   // 将未知的错误继续抛出
      }
    }
  }

  try {
    readData()
  } catch(e) {
    alert("External catch got: " + e)
  }
  ```
  + 在上例中，由于 `readData()` 中的 `catch(err)` 无法处理产生的错误，因此将该错误再次抛出
  + 再次抛出的错误将由外层 `try...catch` 中的 `catch(e)` 捕获，从而对其进行处理

## try...catch...finally
+ `finally {...}` 是 `try...catch` 的一个可选的子句
+ 完成的形式如下：
  ```javascript
  try {
    // try to execute the code
  } catch(e) {
    // handle errors
  } finally {
    // execute always
  }
  ```
+ 当 `finally` 子句存在时，则执行流最后都会流转到 `finally` 的代码块中：
  + 执行了 `try` 之后，没有错误，则跳过 `catch` 执行 `finally`
  + 如果有错误，终止了 `try` 的执行，执行完 `catch` 后，执行 `finally`
+ `finally` 通常用在不管发生何种情况都要采取一定措施的情况下
  + 例如，在以下的斐波那契数列的计算中，需要计算执行的时间，那么有可能用户会输入一个非法的内容，也有可能是正确的，但不管是何种情况，都希望给出运行时间，具体实现如下代码：
    ```javascript
    let num = +prompt('Enter a positive integer number', 35)

    let diff, result

    function fib(n) {
      if (n < 0 || Math.trunc(n) != n) 
        throw new Error('Must not be negative, and also an integer.')
      return n <= 1 ? n : fib(n - 1) + fib(n - 2)
    }

    let start = Date.now()

    try {
      result = fib(num)
    } catch(e) {
      result = 0
    } finally {
      diff = Date.now() - start
    }

    alert(result || 'error occurred')
    alert(`execution took ${diff}ms`)
    ```
+ `try...finally` 结构也是对的，可以没有 `catch` 语句，也就是说不需要对错误进行处理

### finally & return
+ `finally` 适用于 `try...catch` 的任何退出，不包括一个明确的 `return` 语句
+ 例如在下例中，`try` 语句中包含了 `return` 语句，那么一样会先执行 `finally` 中的代码，然后再执行 `return`
  ```javascript
  funciton func() {
    try {
      return 1
    } catch(e) {
      // ....
    } finally {
      alert('finally')
    }
  }

  alert(func())   // finally, 1
  ```
  + 根据执行结果可以看到，会先执行了 `finally` 语句，最后才执行 `return` 语句


## 全局 catch
+ 当 `try...catch` 外发生了错误时，那么可以通过一个全局 catch 来捕获错误
+ `window.onerror` 是浏览器内置的属性，它将在未被捕获的错误的情况下运行
+ 形式如下：
  ```javascript
  window.onerror = function(message, url, line, col, error) {
    // ...
  }
  ```
  + `message`：错误的信息
  + `url`：发生错误的脚本的URL
  + `line, col`：发生错误的行号、列号
  + `error`：错误对象