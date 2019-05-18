## 对象方法中的 this
+ 为了在对象内某个属性可以访问对象的其它属性，一般使用**this**关键字来指向对象本身
+ this 通常用在方法中来指代调用方法的 点 之前的对象
  ```javascript
  let user = {
    name: 'John',
    age: 30,

    sayHi() {
      alert(this.name)
    }
  }

  user.sayHi()  // John
  ```
  + 在上述代码 `user.sayHi()` 的执行过程中，`this` 指代的就是 `user`
  + 当然也可以在不使用 this 的情况下，通过外部变量名来引用它，例如：
    ```javascript
    let user = {
      name: 'John',
      age: 30,

      sayHi() {
        alert(user.name)  // 直接使用 user 来代替 this
      }
    }
    ```
  + 但这样的代码是不可靠的，如果将 user 复制给另一个变量，并对 user 重新赋值，那么对于新变量来说就会访问到错误的内容。例如：
    ```javascript
    // 在上面代码基础上，添加：
    let admin = user;
    user = null  // user 的值已经被改变

    admin.sayHi()   // Error！由于 user 的值已经被改变，所以再通过 user.name 来访问对象的 name 属性是不对的
    ```
  + 所以只有使用 this 来指向对对象的具体引用，才能避免上述所说的情况，this 始终指向当前的对象引用（点号之前的变量）

## this 不受限制
+ this 可以用于任何函数中
+ this 是在**运行时**求值的，可以是任何值，例如，从不同的对象中调用同一个函数可能会有不同的 this 值
  ```javascript
  let user = { name: 'John' }
  let admin = { name: 'Admin' }

  function sayHi() {
    alert(this.name)
  }

  // 在两个对象中使用的是相同的函数
  user.f = sayHi
  admin.f = sayHi

  /** 两个对象在对函数进行调用时，this 有不同的值
   *  函数内部的 this 始终指向引用时 点号 之前的对象
   */
   user.f()   // John (this == user)
   admin.f()  // Admin (this == admin)

   admin['f'] // Admin（不管使用点号还是方括号都没有关系）
  ```
+ 实际上也可以在没有对象调用时，对函数内部使用 this
  ```javascript
  function sayHi() {
    alert(this)
  }

  sayHi()   // undefined
  ```
  + 在严格模式下，该函数中的 this 为 undefined，表示没有任何对象来调用该函数
  + 但是在非严格模式下，this 表示的是全局对象（浏览器中的window）
  + *所以：通常在没有对象的情况下，函数内一般不使用 this；如果函数内有 this，那么通常意味着函数是在对象上下文环境中被调用的*

### 复杂的方法调用可能会失去 this
```javascript
let user  = {
  name: 'John',
  hi()  { alert(this.name) },
  bye() { alert('Bye') }
}

user.hi();   // John

// 现在根据 name 属性来决定调用 user.hi 还是 user.bye
(user.name === 'John' ? user.hi : user.bye)()   // Error!
```
+ 在上述代码中，最后一行的三元运算符的结果是 user.hi，该方法会立刻被第二个 () 调用，但是发生了错误（这是在严格模式下的结果，如果说是非严格模式下，this 指向的是 window 对象，但是结果也不会是程序预期的结果）
+ 这里需要了解 `obj.method()` 调用的原理，`obj.method()` 中有两个操作符：
  1. 首先，点号 `.` 取得这个 `obj.method` 属性
  2. 其后的括号 `()` 调用它
  + 如果把这些操作分离开，那么定义在 method 中的 this 就会丢失
    ```javascript
    // 将赋值与方法定义拆分为两行
    let hi = user.hi
    hi()   // Error! this 未定义（严格模式下）
    ```
  + 为了使对象中的 method 起作用，*点号 `.` 返回的不是一个函数，而是引用类型的值*（这里不大理解啥意思🤦‍）
  + 引用类型的值由三个值组成：`(base, name, strict)`
    + `base` 表示对象本身
    + `name` 表示属性
    + `strict` 表示是否使严格模式下（true 为 严格模式）
    + 例如对于 `user.hi` 在严格模式下为：
      ```javascript
      // 引用类型值
      (user, 'hi', true)
      ```
    + 当在 引用类型上调用圆括号 `()` 时，它们会收到有关该对象及其方法的完整信息，并且设置正确的 this（这里 "this = user")
+ 总的来说，如果对象的方法不能被马上调用，那么 this 就会失去对原对象的引用


### 箭头函数没有 this
+ 箭头函数没有自己的 this，如果在箭头函数中使用 this 作为引用，那么 this 将会指向该箭头函数以外的正常函数被调用时的对象（如果没有，在非严格模式下，则是 window 对象）
  ```javascript
  let user = {
    firstName: 'Ilya',
    sayHi() {
      let arrow = () => alert(this.firstName)
      arrow()
    }
  }

  user.sayHi()  // Ilya (this == user)
  ```

#### 未完待续……