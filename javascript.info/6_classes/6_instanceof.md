# 类的检测：“instanceof”
+ **`inctanceof`** 操作符检查一个对象是否属于某个特定的类，将原型链也考虑在内

## instanceof 操作符
+ 形式为：
  ```javascript
  obj instanceof class
  ```
  + 如果 obj 属于该 Class（或者是原型继承的 Class），则返回 true，否则返回 false
+ 同样作用在构造函数 和 内置的 Class（如 Array） 上
  + 例如通过 instanceof 的方式判断一个对象是不是数组
    ```javascript
    let arr = [1, 3, 4]

    arr instanceof Array    // true
    arr instanceof Object   // true（Array 原型继承自 Object）
    ```
+ 前面学习中提到过，不要随意直接更改 `F.prototype`，例如在以下例子中，`inctanceof` 的结果为 false
  ```javascript
  function Rabbit() {}
  let rabbit = new Rabbit()

  // 修改了 Rabbit.prototype
  Rabbit.prototype = {}

  alert(rabbit instanceof Rabbit)   // false
  ```
  + 因为默认的 `F.prototype.constructor` 指向的是 `F` 本身，所以可以验证其生成的对象实例是属于该 `F` 的，但是在上例中更改了 `Rabbit.prototype = {}` 之后，失去了 `constructor`，所以没办法进行验证

  ## [Bonus: Object.prototype.toString for the type](https://javascript.info/instanceof#bonus-object-prototype-tostring-for-the-type )