# 静态属性和方法
+ 「类的基本语法」该节中提及到：在类中定义的方法，将会被添加到 `Class.prototype` 中，而不是 `Class` 本身，如下例所示：
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
  + 对应的结构图如下：
  ![fig1](https://javascript.info/article/class/class-user.png)
+ 那么「静态方法」指的是类中的方法，是由类本身去调用，而没有添加到 `Class.prototype` 中，例如在上例中，假设 `sayHi` 是一个静态方法，那么只能出现在 `User` 的结构中，而不是 `User.prototype`

## 「static」定义静态方法
+ 使用 `static` 关键字来定义静态方法
  ```javascript
  class User {
    static staticMethod() {
      alert(this === User)
    }
  }

  User.staticMethod()   // true
  ```
+ 相当于将其指定为构造函数的属性，如下所示：
  ```javascript
  function User() {}

  User.staticMethod = function() {
    alert(this === User)
  }
  ```
  + 所以实际上，类也可以像构造函数那样通过点 `.` 的方式来添加静态方法，即：
    ```javascript
    class User {}

    User.staticMethod = function()  { alert(this === User) }
    ```
+ 静态方法只能由类本身（构造函数本身）进行直接调用，并不作用于其生成的特定对象
  > 这是因为生成的对象的 `[[Prototype]]` 引用指向的是 `Class.prototype`，但是静态方法并不存在于 `Class.prototype`，所以生成的对象并不能直接访问到（但是是可以通过访问 `constructor` 来执行静态方法的）
  ```javascript
  class User {
    constructor(name) {
      this.name = name
    }
    static test() {
      alert(this === User)
    }
  }

  let user = new User('John')

  User.test()   // true
  user.__proto__.constructor.test()   // true
  ```

### 工厂方法
+ 静态方法通常用于创建工厂方法，即对于重复性的工作来说，可以在类中创建一个静态方法，以达到每次调用的效果，所以被视为「工厂方法」
  ```javascript
  class Article {
    constructor(title, date) {
      this.title = title
      this.date = date
    }
    static createTodays() {
      return new this("Today's digest", new Date())
    }
  }

  let article = Article.createTodays()
  alert(article.title)    // Today's digest
  ```
  + 从上述代码中可以看到，当每次都创建这样的一个 “Today's digest” 时，则可以直接调用 `Article.createTodays()` 方法即可
  + `createTodays()` 是类的方法，所以 `this == Article`
+ 静态方法也用于与数据库相关的类，以搜索/保存/删除数据库中的条目，如下所示：
  ```javascript
  // 假设 Article 是一个特殊的类，用于管理 articles 数据库的内容
  // 通过静态方法删除 article 中的某条记录
  Article.remove({ id: 123 })
  ```

## 静态属性
+ 静态属性是新的语法特性，所以兼容性不是很好
+ 静态属性也是使用 `static` 关键字来定义
  ```javascript
  class Article {
    static publisher = 'Ilya Kantor'
  }

  Article.publisher     // Ilya Kantor
  ```
+ 也可以直接给类赋值，则不需要 `static` 关键字
  ```javascript
  class Article {}
  Article.publisher = 'Ilya Kantor'
  ```
  + 上述两种方式都是一样的，但是建议用 `static` 来定义，因为通过给类赋值的方式容易被忽略，直接定义在类的内部会更好一点
+ 静态属性与静态方法一样，也只能是由类自己来调用，类生成的特定对象是无法对静态属性进行访问
+ 用不用 `static` 来定义一个类属性是有很大差别的，不用 `static` 则该属性会在 `new` 操作时添加到新生成的对象中，而用了 `static` 则不会
  ```javascript
  class Article {
    editor = 'John'
    static publisher = 'Ilya kantor'
  }

  let article = new Article()

  article.editor    // John
  article.publisher   // undefined
  ```

## 继承问题
+ 静态属性和静态方法是可以被继承的，也就是说子类是可以访问父类的静态属性和方法（注意不是子类的特定对象）
+ 例如在下例中，`Animal.compare` 方法可以通过子类 `Rabbit.compare` 访问：
  ```javascript
  class Animal {
    constructor(name, speed) {
      this.name = name
      this.speed = speed
    }
    run(speed = 0) {
      this.speed += speed
      alert(`${this.name} runs with speed ${this.speed}`)
    }
    static compare(animalA, animalB) {
      return animalA.speed - animalB.speed
    }
  }

  class Rabbit extends Animal {
    hide() {
      alert(`${this.name} hides.`)
    }
  }

  let rabbits = [
    new Rabbit('White Rabbit', 10),
    new Rabbit('Black Rabbit', 5)
  ]

  rabbits.sort(Rabbit.compare)
  rabbits[0].run()    // Black Rabbit runs with speed 5
  ```
  + 通过 `extends` 关键字，`Rabbit` 的 `[[Prototype]]` 引用指向了 `Animal`，所以子类 `Rabbit` 就可以直接访问父类 `Animal` 中所有的静态方法：如下图所示：
  ![fig1](https://javascript.info/article/static-properties-methods/animal-rabbit-static.png)