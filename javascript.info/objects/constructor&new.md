## 构造函数
+ 当希望一个对象可以有多个相同实例的时候，可以通过构造函数来实现
+ 构造函数与常规函数没有太大区别，但是有两个约定：
  + 构造函数的函数名以大写字母开头
  + 只能用 new 操作符来执行构造函数
  ```javascript
  function User(name) {
    this.name = name
    this.isAdmin = false
  }

  let user = new User('Jack')
  alert(user.name)  // Jack
  alert(user.isAdmin) // false
  ```
  + 如上代码所示，当一个函数以像 `new User(...)` 这样执行时，按照以下步骤：
    1. 创建一个空对象，并且赋值给 this
    2. 执行函数体：通常要修改 this，给 this 添加新属性
    3. 返回 this 值  
    换句话说，上述代码可以描述为：
    ```javascript
    function User(name) {
      // this = {} （隐式创建一个 this 空对象）

      // 给 this 添加属性
      this.name = name
      this.isAdmin = false

      // return this （隐式返回this）
    }
    ```
    所以 `new User('Jack')` 相当于创建了以下对象：
    ```javascript
    let user = {
      name: 'Jack',
      isAdmin: false
    }
    ```
+ 当有了构造函数之后，就可以通过一个构造函数获得不同的实体对象，这也是构造函数的主要目的：*实现可重用的对象创建代码*

## 构造函数中的 return
+ 通常来说，构造函数里是没有 return 语句的，因为构造函数的任务就是将所有必要的属性都写到 this 里，然后自动返回这个结果
+ 但是如果说在构造函数中写了 return 语句，则按照以下规则：
  + *如果 return 的是一个 object，那么这个 object 的内容就会 替换掉了 this*
  + *如果 return 的是除了 object 以外的任何一种类型的数据，那么被忽略，返回的是 this*
  + 总的来说就是：构造函数返回的必须是个 object，要么是写在 return 里的，要么是默认的 this（**`return` with an object returns that object, in all other cases `this` is returned**)
  ```javascript
  // 构造函数有 return 语句，并且返回一个对象，则覆盖默认的 this
  function BigUser() {
    this.name = 'John'

    return { name: 'Godzilla' } // return an object
  }

  alert( new BigUser().name )  // Godzilla
  ```
  ```javascript
  // 构造函数有 return 语句，并且返回空内容（或者是基本类型的数据），则返回默认的 this
  function SmallUser() {
    this.name = 'John'

    return; // finishes the execution, return this
  }

  alert( new SmallUser().name )  // John
  ```
  
### 省略构造函数的括号
+ 如果构造函数没有参数，可以在 `new` 操作符后省略构造函数后面的括号
  ```javascript
  let user = new User;  // <---- 省略括号
  // 等同于
  let user = new User()
  ```