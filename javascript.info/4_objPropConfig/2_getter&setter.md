# Getters & Setters 属性

+ 对象有两种类型的属性：「数据属性」和「访问器属性」（accessor properties）
+ 访问者属性 本质上是用于获取和设置值的函数，尽管看起来像是外部代码的常规属性

## Getters & Setters
+ 访问器属性由「getter」和「setter」方法表示
+ 在对象字面量中，它们分别用 `get` 和 `set` 表示
```javascript
let obj = {
  get propName() {
    // getter, 获取 obj.propName 时执行的代码
  },
  set propName(value) {
    // setter，设置 obj.propName = value 时执行的代码
  }
}
```
+ 当读取 `obj.propName` 时，使用 getter；当设置值时，使用 setter
+ 如下例中，给 user 对象新添加一个 fullName 属性，并且其属性值与已有的属性值相关联时，可以用访问器来实现：
  ```javascript
  let user = {
    name: 'John',
    surname: 'Smith',

    // 通过访问器添加 fullName 属性
    get fullName() {
      return `${this.name} ${this.surname}`
    }
  }

  // 像访问普通属性那样访问 fullName
  alert(user.fullName)  // John Smith
  ```
+ 对访问器属性进行读取时，只需要像访问数据属性那样即可，而不需要对其进行函数调用
+ 如果要修改访问器属性，只能通过 setter，如果没有 setter，则赋值操作将会出现错误
  ```javascript
  let user = {
    name: 'John',
    surname: 'Smith',

    get fullName() {
      return `${this.name} ${this.surname}`
    },

    set fullName(value) {
      [this.name, this.surname] = value.split(' ')
    }
  }

  user.fullName = 'Alice Cooper'

  alert(user.name)  // Alice
  alert(user.surname)   // Cooper
  ```
  + 给访问器属性添加了 setter 之后，就可以像数据属性那样进行赋值操作
+ 一个属性名可以是「数据属性」或「访问器属性」但不能同时属于两者，也就是说一个属性名不能同时是「数据属性」和「访问器属性」，否则会发生栈溢出错误
+ 访问器属性只有同时兼有 getter 和 setter，才能可读可写

## 访问器描述符
+ 访问器属性的描述符与数据属性相比，是不同的
+ 对于访问器属性来说，没有 `value` 和 `writable`，但是有 `get` 和 `set` 函数
+ 所以访问器属性的描述符有：
  + **`get`**：一个没有参数的函数，在读取属性时工作
  + **`set`**：带有一个参数的函数，当属性被设置时调用
  + **`enumerable`**：与数据属性相同
  + **`configurable`**：与数据属性相同
+ 如下例所示，使用 `defineProperty` 创建 `fullName` 的访问器，可以使用 get 和 set 来传递描述符
  ```javascript
  let user = {
    name: 'John',
    surname: 'Smith'
  }

  user.defineProperty(user, 'fullName', {
    get() {
      return `${this.name} ${this.surname}`
    },
    set(value) {
      [this.name, this.surname] = value.split(' ')
    }
  })
  ```
  + `enumerable` 和 `configurable` 默认为 false
+ 属性要么是访问器，要么是数据属性，而不能两者都是，所以如果试图在同一个描述符中提供 `get` 和 `value`，则会报错
  ```javascript
  Object.defineProperty({}, 'prop', {
    get() {     // 表示这是访问器
      return 1
    },
    value: 2    // 数据属性才有的
  })
  ```

### 更聪明的 getter/setter
+ getter/setter 可以用作“真实”属性值的包装器，从而可以对它们进行更多的控制
+ 如下例中，如果想禁止为 `user` 设置太短的 `name`，可以将 `name` 存储在一个特殊的 `_name` 属性中，并在 setter 中过滤赋值操作
  ```javascript
  let user = {
    get name() {
      return this._name
    },
    set name(value) {
      if (value.length < 4) {
        alert("Name is too short, need at least 4 characters")
        return
      }
      this._name = value
    }
  }
  user.name = 'Pete'
  alert(user.name)    // Pete
  user.name = ''      // Name is too short...
  ```
  + 当然仍然可以从外部访问 `user._name`，但是一般来说以下划线 "_" 开头的属性被认为是内部的，而不应该从外部进行访问

## 用于兼容性
+ getter/setter 还有一个比较好的应用：它们可以控制普通的的数据属性，并随时调整它们
+ 例如有以下构造函数：
  ```javascript
  funciton User(name, age) {
    this.name = name
    this.age = age
  }

  let john = new User('John', 25)
  ```
+ 但是可能随着代码的构建和需要，想要将 `age` 属性换为 `birthday`
  ```javascript
  funciton User(name, birthday) {
    this.name = name
    this.birthday = birthday
  }

  let john = new User('John', new Date(1994, 9, 1))
  ```
+ 如果修改所有调用处的代码将会是非常大的一项工程，而且有可能还需要去访问 `age`
+ 那么可以考虑通过 getter 来创建一个 `age` 访问属性，这样可以还能访问到对应的值，也不需要修改需要访问的地方
  ```javascript
  funciton User(name, birthday) {
    this.name = name
    this.birthday = birthday

    Object.defineProperty(this, 'age', {
      get() {
        let todayYear = new Date().getFullYear()
        return todayYear - this.birthday.getFullYear()
      }
    })
  }

  let john = new User('John', new Date(1994, 9, 1))
  ```
