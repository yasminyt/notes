## 基本类型中的方法/属性
+ js 中为基本类型提供了相应的方法和属性，但是只有对象才能说在内部定义方法和属性，所以这里涉及到了基本数据类型的内置对象问题
+ 引用类型（object）相比于基本类型（string、number、boolean、symbol、null、undefined）要占用更多的内存，表现出更加重量级的数据特性
+ 基本类型则需要尽可能的轻量，所以相对应的方法和属性不应该是定义在基本类型之上
+ 为此，浏览器定义了这样的一个机制（并且高度优化了以下过程）：
  + 为每一种基本类型定义了对应的对象，在对象内封装了对应的方法和属性
  + 在调用每一种基本类型的方法/属性时，浏览器会根据该基本数据类型创建一个临时的内置对象
  + 调用该临时内置对象的方法/属性，并返回相应的结果
  + 销毁该临时内置对象
+ 所以通过这样的机制，使得基本数据类型也能表现出像引用类型（object）这样的特性
  ```javascript
  let str = 'Hello'

  alert( str.toUppercase() )  // HELLO
  ```
  + 如上述代码中，对于基本类型的 `str` 是如何调用 `toUppercase()` 方法：
    1. `str` 是一个基本的数据类型 string，在访问该数据类型的方法的那一刻，就会创建一个特殊的 object，并且知道其数据值，还有一些可用的方法
    2. 对象中对应的方法被执行，并返回一个新的 string，由 `alert()` 来显示
    3. 销毁这个特殊的 object，只剩下原本的基本类型 `str`

### 通过 String/Number/Boolean 创建基本数据类型
+ 可以通过 `String()`、`Number()`、`Boolean()` 来创建 string/number/boolean 基本类型的数据，它们会把一个值转换成对应的类型
+ 但是不能给上述的构造函数添加 `new` 关键词，否则得到的不是基本类型的数据。由于通过构造函数得到的数据类型均为 object，而不是基本类型，所以不推荐使用 `new String()`、`new Number()`、`new Boolean()` 来创建 string/number/boolean 基本类型的数据（不加 new 的情况下是执行函数，将函数的返回值赋给变量）
  ```javascript
  typeof 1      // number
  typeof new Number(1)  // object
  ```
+ `null` / `undefined` 没有任何的方法

#### example
```javascript
let str = 'Hello'

str.test = 4
alert(str.test)   // undefined
```
+ 在上述代码中，可以给一个字符串添加一个属性（34行）
+ 但是添加的属性是无效的（35行）
+ 这是因为，在给字符串添加属性/方法时，浏览器会创建一个临时的内置对象 object，然后给这个内置对象添加属性（`object.test = 4`）（所以34行并没有报错）。但是并不能访问到，因为这个内置对象在添加完成后就被销毁掉了，所以35行的结果运行出来为 undefined