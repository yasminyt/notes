# class 的基本语法

## “class” 语法
+ class 的基本语法如下：
  ```javascript
  class MyClass {
    // class methods
    constructor() { ... }
    method1() { ... }
    method2() { ... }
    method3() { ... }
    ...
  }
  ```
+ 通过 `new MyClass()` 即可创建一个新对象，并且包含了所列举的所有方法
+ `constructor()` 方法由 `new` 自动调用，因此可以在这里初始化对象
  ```javascript
  class User {
    constructor(name) {
      this.name = name
    }
    sayHi() {
      alert(this.name)
    }
  }

  let user = new User('John')
  user.sayHi()    // John
  ```
  + 当调用 `new User('John')` 时：
  1. 创建一个新的对象
  2. `constructor` 使用给定的参数运行，并为其分配 `this.name`
+ 需要注意的是：在类中各方法之间没有逗号隔开！

## class 是什么
+ 在 js 中，class 实际上就是函数
  ```javascript
  typeof User ==  "function"  // true
  ```
+ `class User {...}` 实际做了以下事情：
  1. 创建一个名为 `User` 的函数，该函数将成为类声明的结果
    + 函数代码从 `constructor()` 中获得（如果没有写该方法，那么假设为空）
  2. 在 `User.prototype` 中存储所有的方法，如 `sayHi`
+ 因此，对于新对象，当调用一个方法时，将会从原型中获取，正如在 「F.prototype」一章所述。那么，新的 User 对象可以访问类方法
  + 可以用下图解释 `class User` 的结果：
    ![fig1](https://javascript.info/article/class/class-user.png)
  + 代码验证如下：
    ```javascript
    class User {
      constructor(name) {
        this.name = name
      }
      sayHi() {
        alert(this.name)
      }
    }

    // class 是一个函数
    typeof User === "function"  // true

    // 或者，更准确来说，是一个构造方法
    User === User.prototype.constuctor    // true

    // 定义的方法在 User.prototype 中
    User.prototype.sayHi    // alert(this.name)

    // 在原型中正好有两个定义的方法
    Object.getOwnPropertyNames(User.prototype)  // constructor, sayHi
    ```

## class 不只是一个语法糖
+ 由于在 js 中 class 本质上是一个构造函数，所以被认为只是个语法糖
+ 但是 class 与构造函数相比，有几点重要的不同：
1. 首先，由类创建的函数由特殊的内部属性标记：`[[FunctionKind]]: "classConstructor"`
  + 与常规函数不同，如果没有 `new` 则无法调用类构造函数
    ```javascript
    class User {
      constructor() {}
    }

    alert(typeof User)    // function
    User()    // Error: Class constructor User cannot be invoked without 'new'
    ```
    + 尽管 `typeof User` 的结果是 `function` 但是却不能像普通函数那样进行调用
  + 另外，在大多数 js 引擎中，要表示一个类构造器的字符串形式时，都会以 `class...` 开头
    ```javascript
    class User {
      constructor() {}
    }

    alert(User)   // class User { ... }
    ```
2. 类方法是不可枚举的。对于所有在「prototype」中的方法，类定义将 `enumerable` 标志设置为 `false`
  + 这是好的，当使用 `for...in` 遍历一个对象时，并不希望获得类方法
3. 类总是使用严格模式，所有在类构造器中的方法都会自动是在严格模式下

## 类表达式
+ 和函数表达式类似，类也可以写成表达式的形式，可以在另一个表达式中定义、传递、返回、赋值等
  ```javascript
  let User = class {
    sayHi() {
      alert('Hello')
    }
  }
  ```
+ 和命名函数表达式类似，类表达式也可以添加一个命名，并且该命名只能被类本身内部可见
  ```javascript
  let User = class MyClass {
    sayHi() {
      alert(MyClass)    // MyClass 只能在类中进行访问
    }
  }

  new User().sayHi()    // 显示 MyClass 的定义
  alert(MyClass)    // 发生错误，MyClass 不能在类的外部被访问到
  ```
+ 甚至可以“按需”动态创建类：
  ```javascript
  function makeClass(phrase) {
    return class {
      sayHi() {
        alert(phrase)
      }
    }
  }
  // 创建一个类
  let User = makeClass('Hello')
  new User().sayHi()    // 'Hello'
  ```

## 类的 getters/setters
+ 在类中创建 getter/setter
  ```javascript
  class User {
    constructor(name) {
      this._name = name
    }
    get name() {
      return this._name
    }
    set name(value) {
      if (value.length < 4) {
        alert('Name is too short')
        return
      }
      this._name = value
    }
  }

  let user = new User('John')
  alert(user.name)    // 'John'

  user = new User('')   // Name is too short
  ```

## 类的属性
+ 类中也可以添加属性，类级属性是新的语法特性，旧的浏览器并不兼容
  ```javascript
  class User {
    name = 'John'

    sayHi() {
      alert(this.name)
    }
  }

  new User().sayHi()  // 'John'
  ```
+ 类属性并不是放在 `User.prototype` 中，相反，它是由 `new` 创建。因此，「类属性永远不会在同一个类的不同对象之间共享」
+ 如果类属性名与 `constructor()` 中声明的属性名一致，那么类属性的值就会被覆盖，因为二者都是在 `new` 时创建