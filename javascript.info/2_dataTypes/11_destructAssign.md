## 解构赋值

### 数组解构
+ 运用解构将数组中的元素便捷地赋值给变量
  ```javascript
  let number = [1, 2, 3]

  let [a, b] = number   // a: 1; b: 2
  ```
+ 使用逗号忽略元素，也就是说对于不想赋值出来的元素，可以用逗号来做占位符
  ```javascript
  let [a, , , b] = [1, 2, 3, 4, 5]   // a: 1; b: 4
  ```
+ 该解构方式适用在任何一种可迭代的数据解构中，如 字符串、Set、Map
  ```javascript
  let [a, b, c] = 'abc'   
  // a: 'a'; b: 'b'; c: 'c'

  let [one, two, three] = new Set(1, 2, 3) 
  // one: 1; two: 2; three: 3

  let [lucy, marry, john] = new Map([['lucy', 20], ['marry', 22], ['john', 20]])
  // lucy: ['lucy', 20]; marry: ['marry', 22]; john: ['john', 20]
  ```
+ 可以赋值给任何指定的变量，例如
  ```javascript
  let user = {}

  [user.name, user.surname] = 'Ilya Kantor'.split(' ')
  ```
+ 在 `Object.entries(obj)` / `map.entries()` 的遍历中使用解构，可以很方便的同时访问 key 和 value
  ```javascript
  let user = {
    name: 'John',
    age: 30
  }

  for (let [key, value] of Object.entries(user))
    alert(`${key}: ${value}`)
  ```

#### The rest '...'
+ `...` 运算符可以获得剩下的元素
```javascript
let [one, two, ...rest] = [1, 2, 3, 4, 5, 6]
// one: 1; two: 2; rest: [3, 4, 5, 6]
```

#### 设置默认值
+ 当所声明的参数超过与之解构的可迭代的变量的数时，则参数会被赋予 undefined
  ```javascript
  let [one, two] = []
  // one: undefined; two: undefined
  ```
+ 在解构赋值中，还可以给变量添加默认值
  ```javascript
  let [one = '1', two = '2'] = [1]
  // one: 1; two: '2'

  let [name = prompt('name?'), surname = prompt('surname?')] = ['Julius']
  ```
  + 默认值可以是任意的内容，包括更复杂的表达式，甚至是函数的调用
  + 如果有匹配的解构值，则忽略默认值，否则使用默认值

### 对象解构
+ 基本形式：**`let {var1, var2} = {var1: value1, var2: value2, ...}`**
+ 对象解构与数组解构有以下不同：
  + 变量用花括号 **`{}`**（数组解构用中括号 **`[]`**）
  + 变量名需要与对象中的 key 对应（所以可以认为是将属性名提出来作为变量），但顺序可以不一致
    ```javascript
    let options = {
      title: 'Menu', 
      height: 200, 
      width: 100
    }
    let { height, width, title } = options
    // height: 200; width: 100; title: 'Menu'
    ```
+ 所以也可以给提出来的属性名另外命名，则原属性仅仅是作为匹配
  ```javascript
  let { width: w, height: h, title } = options
  // w: 200; h: 200; title: 'Menu'（width、height 仅用作匹配，而不再作为变量，所以是没有值的）
  ```
+ 如果变量不符合解构的对象中的任何一个 key，则该变量被赋值为 undefined（不会报错）
+ 也可以像数组解构一样，给变量设置默认值
+ 给变量设置默认值 和 新命名属性 可以同时进行
  ```javascript
  let options = {
    title: 'Menu'
  }
  let { width: w = 100, height: h = 200, title } = options
  // w: 100; h: 200; title: 'Menu'
  ```
+ 也可以使用 **`...`** 运算符获得剩余的属性所构成的新对象（但兼容性不是很好）
  ```javascript
  let { width, ...rest } = options
  // width: 100; rest: { title: 'Menu', height: 200 }
  ```

#### 变量声明与定义分开
+ 在上述数组解构和对象解构的例子中，变量的生命与定义都是在同一行中，但是其实是可以分开的（可以分开就意味着可以对已声明的变量重新赋值，而不是只能用新声明的变量）
  ```javascript
  let title, width, height

  [title, width, height] = ['Menu', 200, 100]
  // title: 'Menu'; width: 200; height: 100
  ```
+ 但是在对象结构中，需要多加一对**小括号**（因为花括号 **`{}`** 单独作为一行的开头时，会被认为是一个独立的代码块）
  ```javascript
  ({ title, width, height } = options)
  ```

### 嵌套的解构
+ 当对象或数组的元素又包含了其它的对象或数组时，可以通过嵌套定义变量来获得对应的元素（其实说白了，就是将想要的元素或 key 的值提取出来）
  ```javascript
  let options = {
    size: { width: 100, height: 200 },
    items: [0, 1],
    extra: true
  }

  let {
    size: { width, height },
    items: [item1, item2],
    title = 'Menu'
  } = options
  // width: 100; height: 200（size 只是用作匹配）
  // item1: 0; item2: 1（items 只是用作匹配）
  // title: 'Menu'
  ```
+ 当需要整体的某个属性时，则只访问到其最外层 key 即可
  ```javascript
  let { size, items } = options
  // size: { width: 100, height: 200 }
  // items: [0, 1]
  ```

### 在函数的参数列表中使用解构获得实参
+ 当函数的参数个数不固定时，并且希望能够给形参设置默认值，仅仅通过 **`arguments`** 关键字是不够的，解构则可以很好的解决该问题
  ```javascript
  let options = {
    title: 'Menu',
    items: [0, 1]
  }

  function func({title = 'Untitled', width = 200, height = 100, items = []}) {
    // ...
  }

  func(options)
  ```
+ 但是通过该方式的话，则只能将所有要传递的参数封装到一个对象中，该对象作为实参，所以说看情况进行选择
+ 另外，如果没有任何参数要进行传递，也必须要放一个空对象，不然会报错
  ```javascript
  func({})  // 传空对象，则函数会使用参数的默认值

  func()  // 报错
  ```