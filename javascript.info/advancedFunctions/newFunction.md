# `new Function` 语法

## 语法介绍
+ `let func = new Function([arg1[, arg2[, ...argN]],] functionBody)`
  + `arg1, arg2, ...` 作为所创建的函数的形参，均为**字符串类型**，并且支持使用「Rest Parameters」，即可以使用 `...` 运算符获得剩余的参数，同样得到的是它们构成的数组
    ```javascript
    let func = new Function('a', 'b', '...rest', 'return rest')
    func(1, 2, 3, 4, 5)   // [3, 4, 5] 
    ```
  + 由于历史原因，形参也可以按逗号分隔符的形式给出，即以下三种形式表现一致：
    + 基础写法：`'arg1', 'arg2', ...` 
      ```javascript
      new Function('a', 'b', 'return a + b')
      ```
    + 逗号分隔：`'arg1,arg2', ...`
      ```javascript
      new Function('a,b', 'return a + b')
      ```
    + 逗号和空格分隔：`'arg1 ,arg2', ...`
      ```javascript
      new Function('a, b', 'return a + b')
      ```
  + 「functionBody」也是使用字符串来表示
+ `new Function` 创建函数的方式是 JS 创建函数的众多方法中不太常用的一种，但是该方法对于函数体简单的情况下的使用非常方便
+ 与其它方法相比，这种方法最大的不同是：*它是在运行时使用描述函数的字符串来创建函数的*
  + `new Function` 允许把字符串变为函数，那么在实际应用中，完全可以从服务器获取并执行一个新的函数
    ```javascript
    let str = // 动态从服务端获取到的代码

    let func = new Function(str)  // 直接根据获取到的代码创建相应的函数
    func()
    ```
  + `new Function` 创建函数的应用场景非常特殊，比如「需要从服务器获取代码」或者「动态地按模版编译函数」，在一般的程序开发中很少使用

## `new Function` 中的闭包
+ 通常，函数会使用一个特殊的属性 `[[environment]]` 来记录函数创建时的环境，它具体指向了函数创建时的词法环境
+ 但是如果使用 `new Function` 创建函数，函数的 `[[environment]]` 并不指向当前的词法环境，而是始终指向**全局环境**
  + 比较下面两段代码：
  ```javascript
  // new Function 在闭包中
  function getFunc() {
    let value = 'test'
    let func = new Function('alert(value)')
    return func
  }
  getFunc()()   // error: value 未定义

  // 普通函数在闭包中
  function getFunc() {
    let value = 'test'
    let func = function() { alert(value) }
    return func
  }
  getFunc()()   // test（变量取自 getFunc 的词法环境）
  ```
+ 这一特性的说明详见：[Closure](https://javascript.info/new-function#closure)