# 类的继承
+ 有以下两个类：
  ```javascript
  // Animal
  class Animal {
    constructor(name) {
      this.speed = 0
      this.name = name
    }
    run(speed) {
      this.speed += speed
      alert(`${this.name} runs with speed ${this.speed}.`)
    }
    stop() {
      this.speed = 0
      alert(`${this.name} stopped`)
    }
  }

  let animal = new Animal('My animal')
  ```
  + 上述代码对应的结构图如下所示：
  ![fig1](https://javascript.info/article/class-inheritance/rabbit-animal-independent-animal.png)

  ```javascript
  // Rabbit
  class Rabbit {
    constructor(name) {
      this.name = name
    }
    hide() {
      alert(`${this.name} hides!`)
    }
  }

  let rabbit = new Rabbit('My rabbit')
  ```
  + 上述代码对应的结构图如下所示：
  ![fig2](https://javascript.info/article/class-inheritance/rabbit-animal-independent-rabbit.png)
+ 对于上述所创建的这两个类来说，它们之间是完全独立，没有关联的
+ 但是 `Rabbit` 应该继承自 `Animal`，也就是说，`Rabbit` 应该以 `Animal` 为基础，可以使用 `Animal` 的方法，并用自己的方法进行扩展

## extends
+ 为了建立两个类之间的继承关系，应该使用 `extends` 关键字实现
+ 如下代码即实现了 `Rabbit` 对 `Animal` 的继承，只需要对 `Rabbit` 类进行更改：
  ```javascript
  class Rabbit extends Animal {
    hide() {
      alert(`${this.name} hides!`)
    }
  }

  let rabbit = new Rabbit('White Rabbit')
  rabbit.run(5)   // White Rabbit runs with speed 5.
  rabbit.hid()    // White Rabbit hides!
  ```
  + 如上所示，`Rabbit` 的代码变得更加简短，不仅默认使用了 `Animal` 的 `constructor()`，而且也可以使用 `Animal` 中的各种方法
+ 实际上，在内部，`extends` 关键字添加了从 `Rabbit.prototype` 到 `Animal.prototype` 的 `[[Prototype]]` 引用，如下图所示：
  ![fig2](https://javascript.info/article/class-inheritance/animal-rabbit-extends.png)
  + 因此如果在当前对象上找不到对应的方法，就会顺着原型进行查找

### extends 后允许任何表达式
+ class 语法允许 `extends` 后的表达式不只是一个类，还可以是其它任意表达
+ 例如在下例中，`extends` 后为一个函数调用：
  ```javascript
  function f(phrase) {
    return class {
      sayHi() {
        alert(phrase)
      }
    }
  }

  class User extends f('Hello') { }

  new User().sayHi()    // Hello
  ```
  + 上述代码中，先执行函数 `f()`，由其返回一个 class
+ 不管 `extends` 后面是怎么样的内容，其执行结果都应该是一个类（如上例，返回了一个类）
+ 这一特性允许我们可以根据不同的条件生成不同的类，提供更灵活的继承

## 重写父类的方法
+ 在有些情景下，父类的方法并不适用于当前子类，那么可以在子类中重写该方法，则覆盖了父类的方法（并没有改变父类的方法，只不过是不会再访问到父类的该方法，因为调用该方法时，已经在子类的原型链中找到）
+ 但是有的时候又不想完全覆盖父类的方法，而是应该「建立在它之上，调整或扩展其它功能」
+ class 提供了 `super` 关键词来引用父类的方法
  + **`super.method(...)`**：调用父类的方法
  + **`super(...)`**：调用父类的 `constructor()` 方法（只能在子类的 `constructor()` 中使用）
+ 如下例所示，在 `Animal` 类的 `stop()` 基础上，重写 `Rabbit` 类的 `stop()`
  ```javascript
  class Rabbit extends Animal {
    hide() {
      alert(`${this.name} hides!`)
    }
    stop() {
      super.stop()    // 先调用 Animal 类的 stop() 方法
      this.hide()     // 再调用自己的 hide() 方法
    }
  }

  let rabbit = new Rabbit('White Rabbit')
  rabbit.run(5)   // White Rabbit runs with speed 5.
  rabbit.stop()   // White Rabbit stopped. White Rabbit hides!
  ```

### 箭头函数没有 super
+ 箭头函数中没有 `super` 关键字，如果在箭头函数中使用了 `super`，则会从外部函数中获取
  ```javascript
  class Rabbit extends Animal {
    stop() {
      setTimeout(() => super.stop(), 1000)    // 1s 后执行父类中的 stop()
    }
  }
  ```
+ 而如果是在常规函数中使用 `super` 则会报错，即方法不能用 `function` 声明的形式来写，只能使用简写形式（也不用箭头形式，因为会使得 `this` 无法正确引用）
  + 例如上述代码如果是用常规函数来写：
    ```javascript
    setTimeout(function() {
      super.stop()
    }, 1000)    // Error
    ```

## 重写 constructor()
+ 根据规范，如果一个类继承自另一个类并且没有 `constructor()` 方法，则会生成以下「空」构造方法
  ```javascript
  class Rabbit extends Animal {
    constructor(...args) {
      super(...args)
    }
  }
  ```
  + 当子类没有自己的构造方法时，则默认将所有获得的参数传递给父类的构造方法中
+ 在给子类添加自己的构造方法时，需要注意两点：
  + 必须要调用 `super(...)`，即父类的构造方法
  + 而且 `super()` 的调用必须发生在子类自定义的属性之前，即 `this` 定义属性之前
+ 这是因为：在 js 中，“继承类的构造方法”和所有其它方法之间存在区别；在继承类中，相应的构造函数由一个特殊的内部属性进行标记：`[[ConstructorKind]]: "derived"`
+ 不同之处在于：
  + 当一个普通的构造函数执行时，会创建一个空对象 `this = {}`，并且将相应的属性进行添加
  + 但是当派生的构造函数运行时，它并不会像普通构造函数那样，而是期望父类的构造函数去完成该工作
+ 所以当构建子类的构造方法时，必须要调用 `super`，否则具有 `this` 引用的对象将不会被创建，那么就会报错
  ```javascript
  class Animal {
    constructor(name) {
      this.name = name
      this.speed = 0
    }
    // ...
  }
  class Rabbit extends Animal {
    constructor(name, earLength) {
      super(name)
      this.earLength = earLength
    }
  }

  let rabbit = new Rabbit('White Rabbit', 10)
  alert(rabbit.name)    // White Rabbit
  alert(rabbit.earLength)   // 10
  ```

## super: internals, `[[HomeObject]]`
+ 当存在多个继承关系时，如果要实现对上几层父类的方法的调用，使用 `this` 很容易产生闭环，从而造成死循环，例如：
  ```javascript
  let animal = {
    name: 'Animal',
    eat() {
      alert(`${this.name} eats.`)
    }
  }
  let rabbit = {
    __proto__: animal,
    eat() {
      this.__proto__.eat.call(this)   // (*)
    }
  }
  let longEar = {
    __proto__: rabbit,
    eat() {
      this.__proto__.eat.call(this)   // (**)
    }
  }

  longEar.eat()   // Error: Maximum call stack size excedded
  ```
  + 在上述代码中，(*) 行和 (**) 行之间构成了一个闭环
  + 因为在执行 `longEar.eat()` 时，(**) 中的 `this.__proto__` 指向了 `rabbit` 对象，而且经过 `call()` 调用，`this = longEar`
  + 那么在转到执行 (*) 时，由于 (**) 中的 `call()` 调用，`this = longEar`，所以 (*) 中的 `this.__proto__` 又指回到了 `rabbit` 本身
  + 而 (*) 中又调用了 `call()`，`call()` 中的 `this = longEar`，所以就造成了死循环
  + 以上过程可以由下图所示：
  ![fig1](https://javascript.info/article/class-inheritance/this-super-loop.png)

+ 像这样的问题，单独使用 `this` 是没法解决的

### `[[HomeObject]]`
+ 为了解决这样的问题，js 为函数添加了更多的内部属性：`[[HomeObject]]`
+ 将函数指定为类或对象方法时，函数的 `[[HomeObject]]` 属性值即为该对象
+ 然后，`super` 使用 `[[HomeObject]]` 来解析父原型及其方法
+ 解决上述问题的方案：
  ```javascript
  let animal = {
    name: 'Animal',
    eat() {           // animal.eat.[[HomeObject]] == animal
      alert(`${this.name} eats.`)
    }
  }
  let rabbit = {
    __proto__: animal,
    name: 'Rabbit',
    eat() {           // rabbit.eat.[[HomeObject]] == rabbit
      super.eat()
    }
  }
  let longEar = {
    __proto__: rabbit,
    name: 'Long Ear',
    eat() {           // longEar.eat.[[HomeObject]] == longEar
      super.eat()
    }
  }

  longEar.eat()   // Long Ear eats. (this == longEar)
  ```
  + 上面代码已经正确执行了原型中的 `eat()` 方法
  + 得益于 `[[HomeObject]]` 机制，一个方法知道它的 `[[HomeObject]]` 并从其原型中获取了父方法


### Example
[Class extends Object](https://javascript.info/class-inheritance#class-extends-object)