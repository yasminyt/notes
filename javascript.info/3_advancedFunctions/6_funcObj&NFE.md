# 函数对象
+ 在 js 中，函数也是对象（函数可以想像成可被调用的「行动对象」）
+ 可以调用函数，还可以把函数当作对象来处理：增删属性，引用传参等

## 函数属性 “name”
+ 函数的名字可以通过属性 `name` 来访问（不管是函数声明还是函数表达式）
  ```javascript
  function sayHi() {
    ...
  }
  alert(sayHi.name)   // sayHi
  ```
+ 如果函数参数的默认值为一个函数，那么这个参数也有 `name` 属性
  ```javascript
  function func(sayHi = function() {}) {
    alert(sayHi.name)   // sayHi
  }
  ```
  + 该特性被称为「上下文命名」，如果函数自己没有提供，那么在赋值时，会根据上下文来推测得到
+ 对象方法也有 `name` 属性
  ```javascript
  let user = {
    sayHi() {
      // ...
    },
    sayBye: function() {
      // ...
    }
  }

  alert(user.sayHi.name)    // sayHi
  alert(user.sayBye.name)   // sayBye
  ```
+ 也有无法推测 `name` 的情况，此时 `name` 属性为空
  ```javascript
  let arr = [function() {}]

  alert(arr[0].name)  // ""（空字符串）
  ```
+ 实际上，大多数函数都是有名字的

## 函数属性 “length”
+ 函数的另一个内置属性 `length`，返回函数形参的个数
  ```javascript
  function f1(a) {}
  function f2(a, b) {}
  function many(a, b, ...more) {}

  alert(f1.length)  // 1
  alert(f2.length)  // 2
  alert(f3.length)  // 2
  ```
+ `length` 不计算余参的个数（从上述代码中可以看到，只计算了明确定义的形参）
+ 可以根据 `length` 的值来对函数作相应的操作

## 给函数自定义属性
+ 可以直接给函数添加自定义的属性
  ```javascript
  function sayHi() {
    alert('Hi')

    // 记录函数运行的次数
    sayHi.counter++
  }
  sayHi.counter = 0

  sayHi() // Hi
  sayHi() // Hi
  alert(sayHi.counter)  // 2
  ```
+ 「属性不是变量」
  + 函数的属性与函数的变量二者毫不相关
  + 因此，函数属性有时可以用来替代闭包，如下可以将闭包中提到的例子改写为用函数属性实现：
    ```javascript
    function makeCounter() {
      // 去掉：let count = 0
      function counter() {
        return counter.count++
      }

      counter.count = 0
      return counter
    }
    let counter = makeCounter()
    alert(counter())  // 0
    alert(counter())  // 1
    ```
  + 那么使用属性还是使用闭包如何取舍：使用属性的一个好处是，可以从函数外部访问到。因此根据实际的情况进行选用

# 命名函数表达式 NFE
+ NFE（Named Function Expression），是具有名称的函数表达式的术语
+ 给一个函数表达再添加函数名，是可以的
  ```javascript
  let sayHi = function func(who) {
    alert(who)
  }
  sayHi('John') // John
  ```
+ 如上代码所示，仍然还是一个函数表达，在 `function` 后面再添加一个函数名，并没有让它变成一个函数声明
+ 像这样给函数表达中的函数再添加一个函数名有以下两点特别的：
  + 它允许函数在内部引用自身
  + 它在函数外部不可见
  ```javascript
  let sayHi = function func(who) {
    if (who) {
      alert(who)
    } else
      func('Guest')   // 引用自身
  }
  sayHi()   // Guest
  func()  // Error, func is not defined（在函数外不可见）
  ```
+ 这样的命名方式的应用场景在于：当需要在函数内部引用自身时，如果给函数一个「函数级」的命名，则不管外部的命名发生任何变化，在函数内部都能正确引用自身（类似于 `this` 的用法）
  ```javascript
  let sayHi = function fun(who) {
    if (who)
      alert(who)
    else
      func('Guest')
  }
  let welcome = sayHi
  sayHi = null

  welcome() // Guest（仍然能正常）
  ```