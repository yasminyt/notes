# 属性标签和描述符

## 属性标签
+ 对象的属性有三个标签，分别用来设定属性的一些特性：
  + **`writable`**：如果为 true，则表示属性可修改，否则为只读
  + **`enumerable`**：如果为 true，则表示可以在循环中被遍历，否则不会出现在循环中
  + **`configurable`**：如果为 true，则可以删除该属性，并且修改这些属性，否则不能
+ 当以一般方式添加对象的属性时，以上标签都为 true，当然也可以随时修改它们

### 访问属性标签
+ 通过 `Object.getOwnProperyDescriptor` 方法查询一个属性所有的信息：
  ```javascript
  let descriptor = Object.getOwnPropertyDescriptor(obj, propertyName)
  ```
  + `obj` 表示要访问的对象
  + `propertyName` 表示要访问的属性的属性名
  + 返回值是一个「属性描述符」对象，包含了属性的值以及所有的标签信息
    ```javascript
    let user = {
      name: 'John'
    }

    let descriptor = Object.getOwnPropertyDescriptor(user, 'name')
    // {value: "John", writable: true, enumerable: true, configurable: true}
    ```

### 修改属性标签
+ 使用 `Object.defineProperty` 修改属性标签：
  ```javascript
  Object.defineProperty(obj, propertyName, descriptor)
  ```
  + `descriptor`：要应用的属性描述符
+ 如果要定义的 `propertyName` 存在，则 `defineProperty` 会更新其标志内容；否则，根据给定的值和标志创建新的 `propertyName`，在这种情况下，如果没有提供标志，则假定为 false（不再是一般定义中默认的 true）
  ```javascript
  let user = {}
  // 给 user 的 name 属性设置 value，由于 user 为空，所以会创建该属性
  Object.defineProperty(user, 'name', { value: 'John' })  
  // 再查看 user 的 name 属性的所有信息
  Object.getOwnPropertyDescriptor(user, 'name')
  // {value: "John", writable: false, enumerable: false, configurable: false}
  ```

### Read-only 属性
+ 通过修改 `writable` 标签的值，使得属性只读
+ 当属性存在时，修改其 `writable`
  ```javascript
  let user = { name: 'John' }

  Object.defineProperty(user, 'name', { writable: false })

  user.name = 'Pete'  // 再修改 name 属性值时，报错
  ```
+ 当属性不存在时，则默认为只读属性
  ```javascript
  let user = {}

  Object.defineProperty(user, 'name', {
    value: 'Pete',
    enumrable: true,
    configurable: true,
    // writable 默认为 false
  })

  user.name = 'Alice'   // 修改 name 的值，报错
  ```

### Non-enumerable 属性
+ 对象的大部分内置方法（如 `toString`）都不会在 `for...in` 中被遍历出来，但是对于自定义的方法，却会被遍历，如下所示：
  ```javascript
  let user = {
    name: 'John',
    toString() {
      return this.name
    }
  }

  for(let key in user)
    console.log(key)    // name, toString
  ```
+ 对于不想被遍历到的属性，可以修改它们的 `enumerable` 标签，那么这些属性就不会再出现在 `for...in` 循环中，就像是内置的一样
  ```javascript
  Object.defineProperty(user, 'toString', { enumerable: false })

  for (let key in user)
    console.log(key)    // name
  ```
+ Non-enumerable 属性也不会被 `Object.keys()` 遍历到

### Non-configurable
+ 当设置 `configurable: false` 时，属性既不可以被删除，也不可以再使用 `defineProperty` 进行修改，相当于已经被固定住了
  + 如 `Math.PI` 是一个只读属性，non-enumerable，并且 non-configurable，所以不能修改 `Math.PI` 的值，也不能修改它的任何标签值
+ 也就是说设置了 `configurable: false` 是一条单向道路，无法再对其修改回来（虽然在非严格模式下，不会有任何的报错，但是不意味着操作是成功的）

## Object.defineProperties
+ `Object.defineProperties` 可以允许一次定义多个属性及其对应的标签值：
  ```javascript
  Object.defineProperties(obj, {
    prop1: descriptor1,
    prop2: descriptor2,
    // ...
  })
  ```
+ 例如：
  ```javascript
  Object.defineProperties(user, {
    name: { value: "John", writable: false },
    surname: { value: "Smith", writable: false },
    // ...
  });
  ```

## Object.getOwnPropertyDescriptors
+ `Object.getOwnPropertyDescriptors(obj)` 允许获取 obj 的所有属性及其对应的标签信息
+ **`Object.getOwnPropertyDescriptors()` 与 `Object.defineProperties` 一起使用，就可以实现对象的「浅拷贝」**（又一种实现浅拷贝的方式）：
  ```javascript
  let clone = Object.defineProperties({}, Object.getOwnPorpertiesDescriptors(obj))
  ```
+ `Object.getOwnPropertyDescriptors(obj)` 获取的所有属性中包括了 symbol 类型的属性

### [For more](https://javascript.info/property-descriptors#sealing-an-object-globally)