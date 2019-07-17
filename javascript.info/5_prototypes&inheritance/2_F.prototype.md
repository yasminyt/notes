# `F.prototype`
+ 可以通过构造函数来创建一个新的对象，如 `new F()`
+ 如果 `F.ptorotype` 是一个对象，那么 `new` 操作符使用它来为新对象设置 `[[Prototype]]`
+ `F.ptorotype` 指的是 `F()` 真正具有的属性，而不是指内部的特殊属性 `[[Prototype]]`
  ```javascript
  let animal = {
    eats: true
  }
  function Rabbit(name) {
    this.name = name
  }
  Rabbit.prototype = animal

  let rabbit = new Rabbit('white rabbit')
  alert(rabbit.eats)    // true
  ```
  + 在上例中，设置 `Rabbit.prototype = animal` 说明：当创建一个新的 `Rabbit` 时，将其 `[[Prototype]]` 指定给 `animal`，如下图所示：
  ![fig](https://javascript.info/article/function-prototype/proto-constructor-animal-rabbit.png)
  + 在上图中，`prototype` 是一个水平的箭头，表示着这是一个常规属性；而 `[[Prototype]]` 是垂直的，表示 `rabbit` 是从 `animal` 继承

### `F.ptorotype` 只能用在 `new F()`
+ `F.prototype` 属性仅在调用执行 `new F()` 时使用，它指定新对象的 `[[Prototype]]`，然后 `F.prototype` 与新对象之间没有任何联系（只是都是指向了同样的内容）
+ 所以，如果在创建之后，`F.prototype` 的值发生更改（`F.prototype = <another object>`），那么 `new F()` 新创建的对象的 `[[Prototype]]` 将具有另一个对象，而原先 `new F()` 创建的对象的 `[[Prototype]]` 仍然还是回原来的，而不是新的那个

## 默认 `F.prototype` & `constructor` 属性
### 默认 `F.prototype`
+ 即使不提供，每个函数都有 `prototype` 属性
+ 默认的 `F.prototype` 为一个对象，其值为 `{constructor: f}`，`constructor` 的值 `f` 又为函数本身，即指向了 `F` 函数
  ```javascript
  function Rabbit() {}

  Rabbit.prototype  // {constructor: f} f: f Rabbit() {}
  ```
  ![fig1](https://javascript.info/article/function-prototype/function-prototype-constructor.png)
  + 可以通过以下代码进行检验：
    ```javascript
    Rabbit.prototype.constructor == Rabbit   // ture
    ```
  + 所以如果不修改 `F.prototype` 的值，那么所有通过构造函数新生成的对象都可以通过自身的 `[[Prototype]]` 访问到 `constructor` 属性
    ```javascript
    let rabbit = new Rabbit()
    rabbit.constructor == Rabbit    // true
    ```
    ![fig2](https://javascript.info/article/function-prototype/rabbit-prototype-constructor.png)
+ 因此，通过 `constructor` 属性也可以利用新生成的对象再构造新的对象，「因为访问 `constructor` 属性得到的就是构造函数本身」
  ```javascript
  let rabbit2 = new rabbit.constructor('black rabbit')
  ```

### `constructor` 属性
+ 「js 本身并不能保证 `constructor` 的值」，也就是说 `constructor` 是可以被修改的
+ 如开篇时所述，可以直接执行 `Rabbit.prototype = animal`，那么这时就不会再有 `constructor` 对象
+ `constructor` 可以让我们判断新生成的对象是由哪一个构造函数生成的，所以不应该直接覆盖 `F.prototype` 的值致使其丢失
  ```javascript
  function Rabbit() { }
  Rabbit.prototype = {
    jumps: true
  }

  let rabbit = new Rabbit()
  rabbit.constructor == Rabbit  // false
  ```
+ 所以通常来说是给 `F.prototype` 「添加属性」，而不是直接覆盖其值
  ```javascript
  function Rabbit() {}

  Rabbit.prototype.jumps = true
  ```
+ 也可以重新添加 `constructor` 到 `F.prototype` 中
  ```javascript
  Rabbit.prototype = {
    jumps: true,
    constructor: Rabbit
  }
  ```