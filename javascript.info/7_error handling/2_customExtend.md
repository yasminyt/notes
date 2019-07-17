# 自定义错误 & 扩展 Error

## 扩展 Error
+ 在自定义错误时，最好是对 `Error` 这个错误的构造函数进行扩展
+ 即通过对 `Error` 类的继承，可以得到自定义错误子类，那么可以使用 `obj instanceof Error` 来识别不同类型的错误对象
+ JS 内置的 `Error` 类的形式如下：
  ```javascript
  class Error {
    constructor(message) {
      this.message = message
      this.name = 'Error'     // 对于不同的内置错误类则有不同的 name 值
      this.stack = <nested calls>   // 非标准的属性，但大部分环境都支持
    }
  }
  ```
+ 在以下代码中，创建了一个 `ValidationError` 的子类，来查看给定的 JSON 数据中是否包含了所需的字段
  ```javascript
  // 创建 ValidationError 子类
  class ValidationError extends Error {
    constructor(message) {
      super(message)
      this.name = 'ValidationError'
    }
  }

  // 使用 ValidationError 来验证
  funciton readUser(json) {
    let user = JSON.parse(json)
    if (!user.age)
      throw new ValidationError('No field: age')

    if (!user.name)
      throw new ValidationError('No field: name')
  }

  let json = `{"name": "John", "age": 30}`
  try {
    let user = readUser(json)
  } catch(err) {
    if (err instanceof ValidationError)       // 判断是否是 ValidationError
      alert('Invalid data: ' + err.message)
    else if (err instanceof SyntanxError)
      alert('JOSN Syntax Error: ' + err.message)
    else
      throw err     // 如果都不属于上述已知的错误，那么就将错误再抛出
  }
  ```
  + 在上述代码中，对错误的类型分别进行判断时，使用了 `instanceof` 关键字
  + 也可以通过 `err.name` 来进行判断，如 `err.name == ValidationError`

## 更进一步的继承
+ 在上述例子中，实现了 `ValidationError` 子类对 `Error` 的继承
+ 在此基础上，还可以对 `ValidationError` 作进一步的细分，即可以实现更细致的验证
+ 在下述代码中进一步实现了 `PropertyRequiredError` 类对 `ValidationError` 的继承，以实现是否已经存在所需的属性
  ```javascript
  // 自定义的 ValidationError 类
  class ValidationError extends Error {
    constructor(message) {
      super(message)
      this.name = 'ValidationError'
    }
  }

  // 进一步创建 PropertyRequiredError 类
  class PropertyRequiredError extends ValidationError {
    constructor(property) {
      super('No property: ' + property)
      this.name = 'PropertyRequiredError'
      this.property = property      // 这个存不存储不要紧
    }
  }

  function readUser(json) {
    let user = JSON.parse(json)

    if (!user.age)
      throw new PropertyRequiredError('age')
    if (!user.name)
      throw new PropertyRequiredError('name')
  }

  try {
    let user = readUser(`{"age": 25}`)
  } catch(err) {
    if (err instanceof ValidationError) {
      alert('Invalid data: ' + err.message)   // Invalid data: No property: name
      alert(err.name)   // PropertyRequiredError
      alert(err.property) // name
    }
    else if (err instanceof SyntaxError) 
      alert(`JSON Syntax Error: ${err.message}`)
    else
      throw err
  }
  ```
+ 对于上述代码，有一个可改进的点，即每一次在定义新的子类时，都使用了 `this.name = <class name>`，实际上，可以在最上层的类通过 **`this.name = this.constructor.name`** 则可以省去重复定义，因为后面的子类会自动继承这一设置
  ```javascript
  // 最上层的子类
  class MyError extends Error {
    constructor(message) {
      super(message)
      this.name = this.constructor.name
    }
  }

  // 建立下一层子类
  class ValidationError extends MyError {}

  // 建立下下一层子类
  class PropertyRequiredError extends ValidationError {
    constructor(property) {
      super(`No property: ${property}`)
      this.property = property
    }
  }

  alert(new PropertyRequiredError("field").name)    // PropertyRequiredError
  ```
  + 这是因为在创建实例时，用哪个构造类来创建，那么 `this.constructor` 就指向的是哪个

