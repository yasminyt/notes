## Computed properties (计算属性)
+ 在 object 中，可以使用 `.` 或者 `[]` 来访问对象中的属性
+ 这里主要是要提一点是说：`[]` 也可以使用在对象字面上，这就称为 计算属性。例如：
  ```javascript
  let fruit = prompt('which fruit to buy?', 'apple')

  let bag = {
    [fruit]: 5    // 将变量使用在对象字面上，也需要通过[]
  }
  alert( bag.apple )
  ```
+ 在对象中，可以使用保留字来作为对象的属性名
  ```javascript
  let obj = {
    for: 1,
    let: 2,
    return: 3
  }
  console.log( obj.for + obj.let + obj.return ) // 6
  ```
  - 所有的保留字（如 let、for、return 等）都可以使用，这是因为在 object 中，所有的属性名都被视为 string 类型
  - 除了 `__proto__`，即便使用它作为对象的属性名，其赋予的属性值也会被忽略掉

## Existence check（检查对象属性）
+ 通过 `in` 操作符检查对象中是否存在某个属性，存在则结果为 true，否则为 false
+ ```javascript
  "key" in object  
  ```

  ```javascript
  let user = { name: 'John', age: 30 }

  alert('age' in user)      // true
  alert('blabla' in user)   // false
  ```
+ 相比于通过 `object[key]` 是否为 `undefined`，来判断一个属性是否在对象中存在，使用 `in` 要更好一些，因为不能排除说给属性赋值为 `undefined` 的情况，如下代码所示：
  ```javascript
  let obj = { test: undefined }

  alert(obj.test)   // undefined（但是属性是存在的，只是值是 undefined，这样的情况下无法判断没有属性）
  alert("test" in obj)  // true
  ```
  - 尽管这种情况非常少见，一般是用 `null` 表示值是未知的或者是空的，但是使用 `in` 会是可以规避一切边缘情况的最好方式

## 对象属性的排序
+ 在列举对象的所有属性时，其属性的顺序是会不一致的，简单来说分两种情况：
  - **如果属性值是整数的字符串，则按照从小到大的顺序来列出**
  - **除了上述所说的情况外，其余类型的属性（其实都是字符串，只是上面所说的字符串是纯数字）都是按照添加时的顺序来列出**
  ```javascript
  let code = {
    '20': 'Germany',
    '25': 'China',
    '1': 'USA'
  }
  for (let key in code) {
    alert(key)    // 1，20，25
  }
  ```
  - 如上代码所示，对于“整数属性”，在遍历对象所有的属性时，是按照它们从小到大的顺序输出（其实这个实际上是和它们在内存中的存储有关），只能是**纯整数**，即便是像 '04' 这样的数字前带 0 的，也会被认为是字符串，因为必须是这个整数没有被改变原来的内容
+ 要使“整数属性”也能按照添加时的顺序列出，只要给每个“整数属性”的数字前添加上 `+` 即可
  ```javascript
  let code = {
    '+20': 'Germany',
    '+25': 'China',
    '+1': 'USA'
  }
  for (let key in code) {
    alert(key)    // 20，25，1
  }
  ```