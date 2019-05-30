# 函数绑定

## 失去 “this”
+ js 中很容易失去 `this` 的引用，例如当方法与对象分开而被传递到某处时，`this` 就会丢失
+ 在 `setTimeout/setInterval` 中执行对象的方法时，很容易发生上述的问题，例如：
  ```javascript
  let user = {
    firstName: 'John',
    sayHi() {
      alert(this.firstName)
    }
  }

  setTimeout(user.sayHi, 1000)  // undefined
  ```
  + 在 `setTimeout` 中，获得的 `user.sayHi` 方法与对象分离了，失去了 `user` 上下文

## Solution1: 包装器
+ 可以为要执行的对象方法添加一层包装器，如下所示：
  ```javascript
  setTimeout(() => user.sayHi(), 1000)
  ```
  + 通过添加一层包装器，则可以将对象方法的执行放在该包装器中，由于方法是立即执行，所以 `this` 始终指向的是 `user` 对象

## Solution2: “bind” 方法
+ 函数提供了一个内置方法 `bind` 来允许修复 `this`
+ 基本形式如下：
  ```javascript
  let boundFunc = func.bind(context)
  ```
  + `func.bind(context)` 的结果是一个特殊的函数，可以作为函数调用，并设置 `this = context`
  + 换句话说，调用 `boundFunc` 就像是调用 `func`，并且固定住了 `this`
+ 则使用 `bind` 解决 `setTimeout` 中的对象方法的 `this` 丢失问题，如下所示：
  ```javascript
  let sayHi = user.sayHi.bind(user) // 将 user.sayHi 中的 this 固定为 user
  setTimeout(sayHi, 1000)   // John
  ```
+ 尽管 `bind` 方法返回的是一个函数，但是并不能对这个特殊的函数再次使用 `bind` 方法，尽在第一次创建时记忆上下文（如果提供了对应的 `context`），例如：
  ```javascript
  function f() {
    alert(this.name)
  }

  f = f.bind( {name: "John"} ).bind( {name: "Pete"} );
  // 控制台中打印 f 时，会看到不能向普通函数那样获得对应的函数体

  f(); // John
  ```
+ `bind` 方法的结果是另一个函数对象，所以原函数作了任何新的修改，都不会作用到这个新的函数上，这是说执行这个新的函数就好像执行了原函数一样