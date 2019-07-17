# 原生原型

## Object.prototype
+ 当输出的是一个空对象时：
  ```javascript
  let obj = {}
  alert(obj)    // "[object Object]"
  ``` 
  + 从代码运行结果来看，依然输出的是一个对象的字符串形式
  + 但是对象本身是空的，那么是如何调用了其内置的 `toString` 方法
+ 当使用花括号 `{}` 定义一个对象时，相当于执行了 `obj = new Object()` 这样的构造函数，而 `Object.prototype` 是一个内置的，包含了多种对象方法/属性的对象：
  ![fig1](https://javascript.info/article/native-prototypes/object-prototype.png)
+ 那么当调用 `new Object()` 或者 创建一个字面对象 `{...}` 时，`obj` 的 `[[Prototype]]` 指向了 `Object.prototype` 的值，如下图所示：
  ![fig2](https://javascript.info/article/native-prototypes/object-prototype-1.png)
+ 所以在上例中，当调用 `obj.toString()` 时，将会从原型链中获得该方法
+ 在 `Object.prototype` 链上再没有其他的 `[[Prototype]]`（已经到了链的顶端）
  ```javascript
  alert(Object.prototype.__proto__)   // null
  ```

## 其他内置的原型
+ 其他内置的对象（如 Array、Date、Function等）也将方法保留在原型中
+ 例如，当创建一个数组 `[1, 2, 3]` 时，内部使用默认的 `new Array()` 构造函数
  + 实际上数组数据是被存储到了一个新的对象中，并且 `Array.prototype` 会成为它的原型并提供方法，这非常节省内存
+ 按照规范，所有内置原型都在顶部有 `Object.prototype`，因此可以认为「一切都继承自对象」，如下图所示：
  ![fig1](https://javascript.info/article/native-prototypes/native-prototypes-classes.png)
+ 可以通过以下代码来检验上述内容：
  ```javascript
  let arr = [1, 2, 3]

  alert( arr.__proto__ == Array.prototype )    // true
  alert( arr.__proto__.__proto__ == Object.prototype )  // true
  alert( arr.__proto__.__proto__.__proto__)   // null
  ```
  + 所以判断一个数据是否是数组类型时，可以通过判断其原型是否是 `Array.prototype` 即可

## 基本类型的原型
+ 对于 string、number、boolean 来说也是有其对应的原型的
+ 虽然上述基本数据类型并不是对象，但是在前面也有提及，当要访问它们的方法/属性时，浏览器会使用内置构造函数 `String`、`Number`、`Boolean` 创建临时包装器对象，在提供了相应的方法之后就会消失
  + 这些对象的方法也存在于原型中，分别是：`String.prototype`、`Number.prototype`、`Boolean.prototype`
+ 特殊值 `null` 和 `undefined` 没有对象包装器，因此它们没有任何方法和属性，也没有对应的原型

## 改变原生原型
+ 原生原型是可以被修改的，例如在 `String.prototype` 上添加方法：
  ```javascript
  String.prototype.show = function () {
    alert(this)
  }

  'BOOM!'.show()  // “BOOM!"
  ```
+ 给原生原型添加属性/方法，可以直接作用在该原型对应的数据类型上
+ 但是这样做是不合适的，因为给原生原型添加的方法/属性在一个应用程序中是全局的，那么容易被覆盖，从而破坏了原有的代码，所以通常来说修改一个本地的原型并不好
+ **「在现代编程中，只有一种情况下修改原生原型是合理的，那就是 polyfilling」**
  + polyfilling 是一种术语，即创建一种替代方法，这种方法已经存在于 js 规范中，但是还没有被当前 js 引擎支持
  + 例如，在下例中，给 String 添加一个 `repeat` 方法，以获得原有字符串经过指定重复次数后得到的新字符串
    ```javascript
    // 先判断该方法是否已存在
    if (!String.prototype.repeat) {
      String.prototype.repeat = function(n) {
        return new Array(n + 1).join(this)
      }
    }

    "La".repeat(3)    // "LaLaLa"
    ```

## 从原型中借用
+ 借用方法很灵活，它允许在需要时混合来自不同对象的功能
+ 例如，对于一个类数组的对象来说，则可以借用 `Array.prototype` 中的方法来对该对象像数组一样操作
  ```javascript
  let obj = {
    0: 'Hello',
    1: 'world',
    length: 2
  }

  // 从 Array.prototype 中借用 join 方法
  obj.join = Array.prototype.join
  
  alert(obj.join)   // "Hello world"
  ```
  + 因为内置的 `join` 方法只关心当前的索引和 `length` 属性，并不会检查数据是对象还是数组
+ 也可以设置 `obj.__proto__ = Array.prototype`，这样在 `obj` 中可以自动使用 Array 中的所有方法