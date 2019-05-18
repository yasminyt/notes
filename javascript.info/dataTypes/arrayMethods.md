## 添加/删除元素
+ `push(...items)`：在数组尾部添加若干元素
+ `pop()`：将数组的最后一个元素移除，并返回该元素
+ `shift()`：将数组的第一个元素移除，并返回该元素
+ `unshift(...items)`：在数组头部添加若干元素

+ **`splice(pos, deleteCount [, elem1, ..., elemN])`**：在指定位置，删除/替换/添加 若干元素
  + `pos` 表示位置，允许为负数，即表示从后往前的位置（从-1开始）
  + `deleteCount` 表示删除的个数
    + 如果为 0，并且有若干个 `elem` 参数，则表示要在 `pos` 位置上**添加**上这些 `elem`；
    + 如果不为 0，并且没有 `elem` 参数，则表示从 `pos` 位置开始，**删除** `deleteCount` 个元素（如果删除的元素个数超出了数组的长度，则删除到尾部即可），并且返回删除的元素构成的数组（即便只有一个元素，返回的也是数组）
    + 如果不为 0，并且有若干个 `elem` 参数，则表示将 `pos` 位置开始的 `deleteCount` 个元素**替换**成 `elem`。替换的本质是上面二者的结合，即会先将 `deleteCount` 个元素删除，然后再在 `pos` 位置上添加这些新的元素，所以替换的元素个数可以超过 `deleteCount`。由于存在删除操作，所以也会返回删除元素构成的数组

+ **`slice(start, end)`**：**获得**从 start 位置开始到 end 位置（不包括 end）的元素，并返回由这些元素构成的数组
  + **`slice()` 不会改变数组本身，所以说只是获得子数组**
  + 数组的 `slice()` 和字符串的 `slice()` 是基本上相同的，只是前者返回的是 子数组，后者返回的是 子字符串

+ **`concat(arg1, arg2, ...)`**：该方法是将给定的若干个数组与原来的数组进行合并
  + 参数的类型可以是：单个元素、数组、对象等
  + 除非是二维数组，否则均按照单个元素的方式在原有数组基础上，在其尾部逐个进行添加
    ```javascript
    let arr = [1, 2, 3]

    arr.concat(4, 5, [6, 7], [[8, 9], 10])  // 1, 2, 3, 4, 5, 6, 7, [8, 9], 10
    ```
  + 添加对象时，也是将整个对象添加上去；但是如果这个对象满足以下几个限制条件，则可以将对应的属性值添加到数组中：
    1. 属性名必须从 0 开始进行标号
    2. 必须有 `Symbol.isConcatSpreadable` 这个属性，并设置为 true
    3. 必须有 length 属性说明有多少个需要添加的元素
    + 实际上按照以上几种限制条件，该对象已经是一个伪数组对象，所以也是类似于将两个数组进行合并
    ```javascript
    let arr = [1, 2]

    // 其实该 obj 已经是一个伪数组 对象
    let obj = {
      0: 'something',   // 属性名必须是用数字
      1: 'else',
      [Symbol.isConcatSpreadable]: true,  //该属性必须设置为 true
      length: 2   // 必须有 length 属性说明有多少个属性
    }

    arr.concat(obj) // [1, 2, 'something', 'else']
    ```
  + `concat()` 不会改变原数组，而是返回一个新的数组

## 数组的遍历
  + **`arr.forEach((item, index, array) => {})`**

## 在数组中进行查找
+ **`arr.indexOf(item [, from])`**：从 from 位置开始查找 item 在 arr 中的位置，如果找到则返回该位置下标，否则返回 -1
  + 如果没有指定 from 参数，则从头开始向后查找
+ **`arr.lastIndexOf(item [, from])`**：从 from 位置 从后往前 查找 item 在 arr 中的位置
  + 如果没有指定 from 参数，则从最后一个元素开始向前查找
+ **`arr.includes(item, from)`**：从 from 位置开始查找 arr 中是否有 item 元素，如果有则返回 true，否则返回 false
  + 如果没有指定 from，则从头开始找
  + from 可以为负数，表示从后面倒数第几个元素开始查找
+ 对以上三种方法的总结&比较：
  + 如果只想知道 arr 是否有某个元素，而不需要知道其具体的位置，那么 `arr.includes` 方法是最好的
  + 均是进行严格匹配（===），所以在匹配过程中不会发生转换
  + 对数组中是否含有 NaN 的查找：
    ```javascript
    const arr = [NaN]

    arr.indexOf(NaN)  // -1
    arr.includes(NaN) // true
    ```
    *由于 === 对 NaN 并不起作用，所以 indexOf 无法进行查找，而使用 includes 则可以进行处理*
+ **`arr.find`**：当数组中包含了比较复杂的元素时（如数组、对象），可以使用该方法对指定条件的元素进行查找
  + 具体形式如下：
    ```javascript
    let result = arr.find((item [, index, array]) => {
      // 如果找到满足条件的元素，则立即停止查找，并返回该元素
      // 如果没找到，则返回 undefined
    })
    ```
  + example
    ```javascript
    let users = [
      {id: 1, name: 'John'},
      {id: 2, name: 'Pete'},
      {id: 3, name: 'Mary'}
    ]

    let user = users.find(item => item.id === 1)

    alert(user)   // {id: 1, name: 'John'}
    ```
+ **`arr.findIndex`**：和 `arr.find` 差不多，只不过它返回的是满足条件的元素的位置下标，而不是元素本身，如果找不到则返回 -1
+ **`arr.filter`**：过滤器

## 数组变换
+ **`arr.map`**：返回新数组
+ **`arr.sort`**：对数组内部的元素进行排序
  + `arr.sort` 默认是将数组中的元素**按照字符串**来进行排序（即逐个逐个字符进行比较），即所有的元素都会先被转换成字符串，然后再进行比较。所以会对数字的排序产生怪异行为
    ```javascript
    let arr = [1, 2, 15]

    arr.sort()
    alert(arr)  // 1, 15, 2
    ```
  + `arr.sort` 中内置的排序算法大部分情况下是，经过优化后的快速排序算法
+ **`arr.reverse`**：对数组中的元素进行倒序处理
+ **`str.split(delim [, length])`**：将字符串按照 delim 规则将其拆分成对应的数组，如果指定了 length，则只会取 length 长度的部分，其余部分被忽略
+ **`arr.join(separator)`**：将数组的所有元素按照指定的分隔符 separator 连接成一个字符串
+ **`arr.reduce/reduceRight`**：基于数组的所有元素来计算得到一个简单结果（例如对数组的所有元素进行求和）
  + 一般形式如下：
    ```javascript
    let result = arr.reduce((previousValue, item, index, array) => {
      // ...
    }, initial)
    ```
  + **`previousValue`**：上一次函数调用的结果，在第一次调用时，如果定义了 initial 的值，则为 initial 的值，否则就以第一个元素作为其值
  + `item`：当前元素（在第一次函数调用时，如果定义了 initial，则 item 为第一个元素的值，否则为第二个元素）
  + **`initial`**： 初始值，如果定义了则作为在第一次调用时 previousValue 的值，但是建议*总是添加上该初始值，以防数组为空时，函数调用发生错误*
  + 例如：
    ```javascript
    let arr = [1, 2, 3, 4, 5]

    let result = arr.reduce((sum, current) => sum + current, 0)
    alert(result) // 15
    ```
    + 上述过程可以用下表描述：

      | 第n次调用 | sum | current | result |
      | -------- |  ----- |  ----- |  ----- |
      | 1 | 0 | 1 | 1 |
      | 2 | 1 | 2 | 3 |
      | 3 | 3 | 3 | 6 |
      | 4 | 6 | 4 | 10 |
      | 5 | 10 | 5 | 15 |

  + `arr.reduceRight` 的用法与之相同，只不过是从右往左遍历
+ *总结：在以上方法中，`sort` 和 `reverse` 会改变数组自身*

## 判断是否是数组
+ **`Array.isArray(value)`**：此为 ES6 的新语法，可以用来判断 value 是否是 数组
  + 因为数组也是对象，所以没办法通过 typeof 来判断是不是数组
+ **原型链**的方式：通过判断原型链上的构造方法是不是 Array，来判断是不是数组
  ```javascript
  arr.__proto__.constructor == Array
  // 如果是数组，则返回 true，否则返回 false
  ```
+ 通过 **`instanceof`** 关键字判断是否是 Array 的实例
  ```javascript
  arr instanceof Array
  ```

## `Array.from`
+ `Array.from` 方法用于将一个类数组的对象转换成一个真正的数组
+ 类数组对象（array-like obj）指的是：一个数组的属性是从 0 开始的下标表示，并且有 length 属性。如下所示：
  ```javascript
  let obj = {
    0: 'a',
    1: 'b',
    2: 'c',
    length: 3
  }
  ```
+ `Array.from(arrayLike [, mapFn, thisArg])` 可以将上述的对象转换成一个真正的数组
  ```javascript
  Array.from(obj) // ['a', 'b', 'c']
  ``` 
  + `arrayLike` 参数可以是一个类数组的对象，也可以是一个数组。但是如果是别的其它类型的数据或不是类数组对象，则返回一个空数组 `[]`
  + `mapFn` 参数提供了一个可选的 map 方法，相当于对新生成的数组作 map 操作，并返回最后的数组
    ```javascript
    Array.from(obj, (item, idx) => item + idx)  // ['a0', 'b1', 'c2']
    ```

## 不同遍历方法对数组空位的处理
+ **数组空位** 指：数组的某一个位置没有任何值，例如：由 Array 构造函数返回的数组都是空位，即
  ```javascript
  let arr = new Array(3)
  // 等同于 let arr = [,,,]
  ```
  + 在上述代码中，`new Array(3)` 返回一个具有 3 个空位的数组
  + **空位不是 undefined，一个位置的值等于 undefined，依然是有值的，空位是没有任何值，使用 `in` 运算符来说明该问题**（`in` 运算符用来对象中是否存在某个属性，数组也是对象，它的 key 就是每个元素对应的下标）
    ```javascript
    0 in [undefined, undefined, undefined]  // true
    0 in [, , ,]  // false

    0 in new Array(5)   // false 
    ```
  + *在访问空数组时，之所以得到每个元素的值为 undefined，是因为经过了转换，而不是元素的值为 undefined*
+ `forEach()`、`filter()`、`every()`、`some()` 都会**跳过**空位
  ```javascript
  let arr = new Array(5), /* 等同于 arr = [,,,,,]（可以认为有多少个逗号就有多少个元素）*/
      arrNull = new Array(5)
  
  arr[2] = 1    // arr = [,,1,,,]
  
  // forEach()
  arr.forEach(item => console.log(item))  // 1

  // filter()
  arr.filter(item => item > 0)    // 1

  // every()
  arrNull.every(item => typeof item == 'undefined') // true （这里的结果居然为 true，true 的话并没有跳过空位。。🤔）

  // some()
  arrNull.some(item => typeof item == 'undefined')  // false （这里给出了与 every 完全相反的结果，按照这个结果的话，some 是跳过了空位）
  ```
+ `map()` 会跳过空位，但是会**保留这个空位**
  ```javascript
  arr.map(item => 'a')   // [,,'a',,,]  
  ```
  + 从以上代码中可以看到，`map()` 虽然跳过了空位，但是仍然保留了原空位的位置，这保证了 `map()` 的特性：即 `map()` 得到的新的数组的长度与原数组的长度是一致的
+ `join()` 和 `toString()` 会将空位视为 undefined，而 undefined 和 null 会被处理**空字符串**
  ```javascript
  arrNull.join('.')   // "...." （即得到了一串连续的省略号）
  arrNull.toString()  // ",,,," （数组使用 toString 转化为字符串，默认是用逗号作分隔符）

  arr.join('.')   // "..2.."
  ```

## Example 
1. *在不使用 loop 循环的情况下，创建一个长度为 100 的数组，并且每个元素的值等于它的下标*
  > **`Array.from()`** 方法的使用
  ```javascript
  Array.from(new Array(100), (item, idx) => idx)
  ```
