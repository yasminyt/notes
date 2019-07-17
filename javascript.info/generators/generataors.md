# Generators
+ 函数的返回值通常只有一个或者没有
+ Generator 函数则可以返回任意多个值，一个或者无穷多个，通过配合 iterables，则可以创建一个数据流

## Generator Functions
+ 生成器函数用 **`function*`** 来表示，语法如下：
  ```javascript
  function* generatorSequence() {
    yield 1;
    yield 2;
    return 3
  }
  ```
+ 当调用这样的生成器函数 `generatorSequence` 时，并不会立即函数内的代码，而是返回一个特殊的对象，可以叫做「生成器」
  ```javascript
  // 生成器函数创建了一个生成器对象
  let generator = generatorSequence()
  ```
  + 生成器对象可以被视为“冻结函数调用”

  ![fig1](https://javascript.info/article/generators/generateSequence-1.png)

  + 创建该对象时，函数内的代码就会暂停在第一行
+ `generator` 的主要方法为 `.next()`，该方法被调用时，就会执行函数内的代码，直到遇到第一个 `yield <value>` 表达，然后停止执行，并且返回这个 `value`
  + 例如在以下代码中，返回了第一个 `yield` 表达的值
    ```javascript
    function* generatorSequence() {
      yield 1;
      yield 2;
      return 3
    }

    const generator = generatorSequence()
    const first = generator.next()

    first   // {value: 1, done: false}
    ```
+ `.next()` 的返回值是一个对象，包括了两个属性：
  + `value`：`yield` 的值
  + `done`：如果函数内的代码还没有执行完，则返回 `false`，否则返回 `true`
+ 在上述示例中，所以只获得了 `1` 的值，继续调用 `generator.next()` 则会获得 `{value: 2, done: false}` 的结果；再继续调用 `generator.next()`，此时函数体内已经遇到 `return` 表达式，所以得到 `{value: 3, done: true}`，此时 `done: true`，函数执行结束
+ 当 `generator.next()` 已经执行完毕时，不管调用多少次都得到 `{done: true}` 的结果，而且也没办法“回滚”一个 generator，也就是回到最开始的时候，但是可以通过调用 `generatorSequence()` 新建一个 `generator`
+ `function* f(...)` 和 `function *f(...)` 的写法都是正确的，但是前者更符合语意，表示这是一个生成器函数

## Generators are iterable
+ 生成器返回的值是可迭代的，可以通过 `for...of` 进行遍历
  + 例如使用 `for...of` 对上例的 `generator` 的结果进行遍历
    ```javascript
    for (let value of generator)
      alert(value)    // 先输出 1，然后输出 2
    ```
+ 需要注意的是，`for...of` 并不会遍历 `return` 的结果，因为当 `done: true` 时，就忽略了这个值的输出，所以在上述代码中，没有输出 3；要解决这一问题，只需要将所有要遍历的结果全部用 `yield`，而不是 `return`
  ```javascript
  function* generatorSequence() {
    yield 1;
    yield 2;
    yield 3
  }
  const generator = generatorSequence()
  for (let value of generator)
    alert(value)    // 依次输出：1、2、3
  ```
+ 由于 `generator` 是可迭代的，所以也可以使用 `...` 扩展运算符来获得 `generator` 的值
  ```javascript
  function* generatorSequence() {
    yield 1;
    yield 2;
    yield 3
  }

  const sequence = [0, ...generatorSequence()]
  console.log(sequence)   // [0, 1, 2, 3]
  ```

## 嵌套生成器
+ 在一个生成器中可以嵌套另一个生成器，通过 `yield* <another generator>` 实现
+ 例如在以下例子中，通过嵌套生成器，则简洁的实现了 0-9、a-z、A-Z 的输出
  ```javascript
  function* generateSequence(start, end) {
    for (let i = start; i <= end; i++)
      yield i
  }

  function* generatePasswordCodes() {
    // 0-9
    yield* generateSequence(48, 57)   // 使用的是 ASCII 码
    // A-Z
    yield* generateSequence(65, 90)
    // a-z
    yield* generateSequence(97, 122)
  }

  let str = ''
  for (let code of generatePasswordCodes()) {
    str += String.fromCharCode(code)
  }   // str 的结果即为 0-9、a-z、A-Z 构成的字符串
  ```
+ `yield*` 将执行委托给另一个生成器，即由 `generateSequence` 来完成，所以 `generatePasswordCodes` 中的代码也等同于：
  ```javascript
  function* generatePasswordCodes() {
    for (let i = 48; i <= 57; i++)  yield i;

    for (let i = 65; i <= 90; i++)  yield i;

    for (let i = 97; i <= 122; i++) yield i;  
  }
  ```

## “yield” 的另一种用法
+ `yield` 不仅可以将结果返回到外部，还可以传递生成器内的值
+ 通过调用 `generator.next(arg)`，其中包含了一个参数，该参数将作为 `yield` 的结果
  ```javascript
  function* gen() {
    const result = yield "2+2?"

    alert(result)
  }

  let generator = gen()
  let question = generator.next().value   // 2+2?
  
  generator.next(4)
  ```
  1. 在上述代码中，第一次调用 `generator.next().value` 时，则执行了 `gen()` 中 `yield "2+2?"` 这一行代码，并且停止执行，返回 `yield` 中的值 `2+2?`，所以 `question` 的结果为 `2+2?`
  2. 第二次调用 `generator.next(4)` 时，在 `gen()` 停止的位置继续执行，将 `4` 传递给了 `result`，所以 `alert()` 出来的结果为 4
  + 上述过程可以用下图表示：

  ![fig1](https://javascript.info/article/generators/genYield2.png)

+ 更复杂一点的例子来说明：
  ```javascript
  function* gen() {
    const ask1 = yield "2+2?"
    alert(ask1)

    const ask2 = yield "3*3?"
    alert(ask2)
  }

  const generator = gen()
  alert(generator.next().value)   // 2+2?
  alert(generator.next(4).value)  // 3*3?
  alert(generator.next(9).done)   // true
  ```
  + 上述代码的执行过程可以由下图表示：

  ![fig2](https://javascript.info/article/generators/genYield2-2.png)

## generator.throw
+ `generator.throw` 可以传递一个异常给 `yield`
  ```javascript
  function* gen() {
    try {
      let result = yield "2+2?"   // (1)

      alert("当产生异常时，这行代码永远不会被执行")
    } catch(err) {
      alert(err)
    }
  }

  const generator = gen(),
        question = generator.next().value
  
  generator.throw(new Error("The answer is not found in my database")))   // (2)
  ```
  + 在上述代码中，（1）处的代码，即 `yield` 传递了（2）处传递的异常，所以 `result` 即为一个 `Error` 对象
  + 由于 `result` 的值为一个异常，所以被 `catch` 捕获
  + 如果这里没有使用 `try...catch` 捕获异常，那么也可以在调用 `generator.throw` 处捕获，即
    ```javascript
    try {
      generator.throw(new Error("The answer is not found in my database")))
    } catch(err) {
      alert(err)    // 也捕获了产生的异常
    }
    ```