# 原型方法 & 对象没有 `__proto__`
+ 前面提到了设置原型的方法可以通过 `__proto__`，但是 `__proto__` 被认为是过时的而且不太合适用在编程中（应该只是作为 js 中的浏览器部分，用作显示原型）
+ 现代的方法是：
  + **`Object.create(proto[, descriptors])`**：使用给定的 `proto` 作为 `[[Prototype]]` 来创建一个「新的对象」，以及可选的属性描述符 `descriptors`
  + **`Object.getPrototypeOf(obj)`**：返回 `obj` 的 `[[Prototype]]` 
  + **`Object.setPrototypeOf(obj, proto)`**：设置 `obj` 的 `[[Prototype]]` 为 `proto`
+ 应该使用上述提及到的方法来设置 `[[Prototype]]`，而不是 `__proto__`
  ```javascript
  let animal = {
    eats: true
  }

  // 创建一个空对象，并且将 animal作为其原型
  let rabbit = Object.create(animal)

  alert(rabbit.eats)    // true
  alert(Object.getPrototypeOf(rabbit) === animal)   // true

  // 修改 rabbit 的原型
  Object.setPrototypeOf(rabbit, {})
  ```
+ `Object.create` 提供了第二个可选参数 `descriptors`，可以给新对象添加对应的属性（按照前面其它章节中提到的描述符内容来写即可）
  ```javascript
  let animal = {
    eats: true
  }

  // 创建一个空对象，设置 animal 为其原型，并且添加 jumps 属性
  let rabbit = Object.create(animal, {
    jumps: {        // 按照描述符的写法来写，所以也可以设置其它描述属性
      value: true
    }
  })

  rabbit    // {jumps: true}
  ```
+ 更简便和有效的「浅拷贝」的方式：
  ```javascript
  let clone = Object.create(Object.getPrototypeOf(obj), Object.getOwnPropertyDescriptors(obj))
  ```
  + 该方法可以将 `obj` 中的所有属性（可遍历的或者是不可遍历的）、数据属性、以及 setters/getters、和正确的 `[[Prototype]]`，所有都可以进行拷贝
  + 该方法实现的依然是浅拷贝而已