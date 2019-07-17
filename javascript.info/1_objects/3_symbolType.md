## Symbol 数据类型
+ object 的 key 的数据类型只能有两类：string 和 symbol，而不是 Number、boolean
+ Symbol 是一个基本类型，用来表示一个**唯一**的标识符
+ 可以通过 `Symbol()` 来创建 Symbol 数据类型，不能加 `new`，因为这个只是一个基本数据类型，而不是创建一个对象   
  ```javascript
  let id = Symbol()
  ```
+ 也可以给每个 symbol 添加一个描述符，也可以叫做 symbol 的名字，仅仅只是个描述符，而不代表值，常常用于标识每一个 symbol，为了方便debugging   
  ```javascript
  // id is a symbol with the description "id"
  let id = Symbol('id')
  ```
+ Symbol 最大的特性就是它的**唯一性**，即便创建了多个具有相同描述的 symbol，它们也是不一样的（描述符仅仅只是一个label，而不会影响其它任何）   
  ```javascript
  let id1 = Symbol('id')
  let id2 = Symbol('id')

  alert(id1 == id2)   // false
  ```
### Symbol 不会自动转换成一个字符串
+ js 中的大多数值都支持隐式转换为字符串，比方说，基本上可以 `alert()` 出任何值，但是 Symbol 是个例外，并不会自动转换成一个字符串    
  ```javascript
  let id = Symbol('id')

  alert(id) // TypeError: Cannot convert a Symbol value to a string
  ```
+ 这是因为 string 和 symbol 根本不同，不应偶尔将其转换为另一个，如果真的想要查看一个 symbol，可以有两种方式：
  + symbol 调用 `.toString()`
    ```javascript
    alert(id.toString())  // Symbol(id)
    ```
  + 通过 `symbol.description` 属性仅仅获得一个 symbol 的描述
    ```javascript
    alert(id.description)  // id
    ```

## 隐藏属性
+ symbol 可以允许在对象中创建“隐藏“属性
  ```javascript
  let user = { name: 'John' }
  let id = Symbol('id')

  user[id] = 'ID value'
  alert(user[id])  // ID value
  ```
+ symbol 类型的变量作为属性 vs string 类型的变量作为属性
  + 由于 `Symbol()` 创建的每个 symbol 具有唯一性，因此当它作为 object 的属性时，就不会被其它代码所覆盖或重写，即便它们拥有相同的描述符，也是不一样的两个 symbol
  + 而 string 类型的属性，会被重写或是覆盖
+ 使用 symbol 数据类型作为属性时，只能时刻用 '`[]`' 来定义和访问属性
  ```javascript
  let id = Symbol('id')

  let user = {
    [id]: 123,
    id: 456
  }
  alert(user[id])   // 123  访问的是 symbol 类型的的属性
  alert(user['id']) // 456  访问的是 string 类型的属性（需要加上引号，不然就会被视为是一个变量
  ```
  *Note：要使用 `Symbol()` 创建的 symbol 来作为属性，一定要先存在一个变量中（像上述代码所示），不然后面就没办法访问这个属性了*
 
### symbol 属性被 `for...in` 跳过
+ symbol 类型的属性并不参与 `for...in` 循环
  ```javascript
  let id = Symbol('id')
  let user = {
    name: 'John',
    age: 30,
    [id]: 123
  }

  for (let key in user) 
    alert(key)    // name, age (no symbols)
  ```
+ 这是一般的“隐藏”概念的一部分，如果其它的脚本或库想要遍历我们的对象时，都无法访问到 symbol 类型的属性
+ 但是，`Object.assign()` 方法会对所有的属性进行拷贝，包括 string 类型的属性和 symbol 类型的属性
  ```javascript
  let id = Symbol('id')
  let user = { [id]: 123 }

  let clone = Object.assign({}, user)
  alert(clone[id])  // 123
  ```

## 全局 symbol
+ 全局 symbol 说的是，symbol 尽管是唯一性的，但是有的情况下（比方说，能够在应用的不同地方都能访问到同一个 symbol），希望能够对具有相同 name 的 symbol 进行访问，则需要一个 *global symbol registry*（全局符号注册表）
+ 通过将 symbol 创建在全局符号注册表中，就可以在随后访问它们，并且保证了重复访问具有相同 name 的 symbol 返回的都是同一个（name 为每一个 symbol 的唯一标识符）
### `Symbol.for(key)`
+ 通过 `Symbol.for(key)` 来创建全局 symbol
+ `Symbol.for(key)` 在创建全局 symbol 时，会先检查全局注册表中是否已经存在了 name 为 key 的 symbol，如果是，则返回这个 symbol，否则，就创建一个 name 为 key 的 symbol，并存在全局注册表中
  ```javascript
  // read from the global registry
  let id = Symbol.for('id')   // if the symbol did not exist, it is created

  // read it again
  let idAgain = Symbol.for('id')

  // the same symbol
  alert(id === idAgain) // true
  ```
### `Symbol.keyFor(sym)`
+ 通过 `Symbol.keyFor(sym)` 获得全局 symbol 中的 name
  ```javascript
  let sym = Symbol.for('name')
  let sym2 = Symbol.for('id')

  // get name from symbol
  alert(Symbol.keyFor(sym))   // name
  alert(Symbol.keyFor(sym2))  // id
  ```
> **`Symbol.keyFor` 只是在 *全局注册表* 中查找 symbol 的 name，对于由 `Symbol()` 创建的非全局的 symbol，是不起作用的，返回 undefined**  
  ```javascript
  alert( Symbol.keyFor(Symbol.for('name')) ) // name, global symbol
  alert( Symbol.keyFor(Symbol('name2')) ) // undefined, the argument isn't a global symbol
  ```

  ## [Summary](https://javascript.info/symbol#summary)