## Map
+ Map 类型的数据也是键值对，和 object 类似
+ 但是 Map 的属性可以是**任意类型**的数据（而 object 的属性只能是 string 类型）
+ 主要的方法有：
  + **`new Map()`**：创建 map
  + **`map.set(key, value)`**：添加新的 map 数据项
  + **`map.get(key)`**：根据 key 返回对应的 value，如果这个 key 不存在，则返回 undefined
  + **`map.has(key)`**：判断 key 是否存在，如果存在返回 true，否则返回 false
  + **`map.delete(key)`**：根据 key 删除对应的 value，如果删除成功则返回 true，否则返回 false（key 不存在）
  + **`map.clear()`**：清空 map
  + **`map.size`**：返回 map 中的元素总和

### 使用 object 作为 map 的 key
+ 使用对象作为键，是 Map 最显著和最重要的功能之一
  + 例如，以下的小 demo 展示了使用 object 作为 key 的作用
  ```javascript
  // 以下使用 map 结构来记录每一个访客为第几位
  let john = { name: 'John' }

  let visitsCountMap = new Map()

  visitsCountMap.set(john, 123)

  // 而传统的使用 object 的做法为：
  let john = { name: 'John', id: 1 }  
  let visitsCount = {}
  
  visitsCount[john.id] = 123  // 需要使用一个标识来记录对应的人
  ```
  + 从以上两种方式可以看到：使用 obj 来直接作为 map 的 key，可以使得表达更优雅，也更简洁
+ Map 中对 key 的匹配与严格比较（===）差不多，唯一的区别是，`NaN` 是被认为相等的（严格和非严格比较中，`NaN` 不等于任何，包括它本身），因此 `NaN` 也可以被用来当 key

### map 的创建
+ **`new map()`** 可以不带参数，则创建一个空的 Map 数据结构；也可以带参数，但是必须是一个二维数组
  ```javascript
  let map = new Map([
    ['1', 'str1'],
    [1, 'num1'],
    [true, 'bool1']
  ])


  map.get('1')   // "str1"
  map.get(1)     // "num1"
  // 可以看到在 map 中，不会自动将 key 转换成字符串，定义的时候是什么数据格式就是什么数据格式
  ```
+ 从对象中创建，通过 **`Object.entries(obj)`** 方法将对象转换成对应的二维数组，从而作为 `new map()` 的参数
  ```javascript
  let map = new Map(Objcet.entries({
    name: 'John',
    age: 30
  }))

  // Object.entries 会返回对象对应的二维数组：[['name': 'John'], ['age', 30]]
  ```
+ 通过对 **`map.set(key, value)`** 的链式调用，为 map 添加元素，由于 `map.set` 返回的是 map，所以可以进行链式调用
  ```javascript
  map.set('1', 'str1')
     .set(1, 'num1')
     .set(true, 'bool1')
  ```

### map 的遍历
+ **`map.keys()`**：遍历 map 中所有的 key
+ **`map.values()`**：遍历 map 中所有的 value
+ **`map.entries()`**：遍历 map 中所有的键值对，并返回每一组 [key, value]，也是 `for...of` 中默认的遍历方式
```javascript
  let numbers = new Map([
    ['one', 1],
    [2, 'two'],
    [3, 'three']
  ])

  for (let key of numbers.keys())
    alert(key)  // 'one', 2, 3

  for (let value of numbers.values())
    alert(value)  // 1, 'two', 'three'

  for (let number of numbers) //等效于 numbers.entries()   
    alert(number) // （逐个显示每个数组的内容）
```
+ 迭代出来的元素的顺序和初始加入时一样，即便 key 是数字，而且是乱序的，也保留了初始添加时的顺序，这一点与 object 不同
+ **`map.forEach()`**：map 也有内置的 forEach 方法，和数组类似，也可以实现对元素的遍历
  ```javascript
  numbers.forEach((key, value) => alert(`${key}: ${value}`))
  ```

## Set
+ set 是值的集合，但是这个集合里每个值都是唯一的，也就是说没有重复值
+ 主要的方法有：
  + **`new Set(iterable)`**：创建一个 set，可选地可以从数组中的 values 中创建（只要是可迭代的都可以）
  + **`set.add(value)`**：添加一个元素，并返回 set 本身（所以也可以像 map 那样链式添加元素）
  + **`set.delete(value)`**：删除一个元素，删除成功返回 true，否则返回 false
  + **`set.has(value)`**：存在返回 true，否则返回 false
  + **`set.clear()`**：清空 set
  + **`set.size`**：返回 set 中的元素个数
+ set 对插入的数据的唯一性检查进行了非常好的优化，所以可以用在对数组去重上

### Set 的遍历
+ 由于 Set 中只有一个值，所以使用 `for...of` 进行遍历即可
+ Set 也有 forEach、keys()、values()、entries() 方法，类似于 map，但是由于 Set 中 key 和 value 都是同一个，所以 keys() 和 vlaues() 的结果是一样的，entries() 得到的是 [value, value]

## WeakMap & WeakSet
+ WeakSet 是一种特殊的 Set，不会阻止 gc 自动回收其中不可达的 items，从而避免内存泄漏。WeakMap 同理
+ 在 weakMap 和 weakSet 中，key 都只能是 object 类型，而不能是其它基本类型的数据，否则会报错
+ 当 key 不可达时，则垃圾回收机制会将其进行回收（但是在控制台上看不出效果，所以不确定这么理解对不对😂）
+ weakMap vs Map
  + weakMap 是针对 key 为 object 时的 Map 的一种扩展
  + 因为当 key 为 object 时，map.keys() 遍历得到的是对具体的对象的引用，而不是指向它们的变量
    ```javascript
    let a = { num: 0 },
        b = { num: 1 },
        map = new Map()

    map.set(a, 'a')
       .set(b, 'b')

    for (let key of map.keys())
      console.log(key)
    // { num: 0 }
    // { num: 1 }
    ```
  + 所以即便当 `a = null; b = null` 时，即 a、b 都分别失去了对对象的引用，它们原本所指的对象也已经被 map 所引用，所以不会被gc回收，即便已经没办法再去调用它，所以就容易造成内存泄漏
  + weakMap 则是为了解决上述问题的一种数据结构。weakMap 结构的数据，当其中的对象失去引用时，则该对象会被从内存和 weakMap 中自动删除
+ weakSet 与上述类似
+ 对于 weakMap 和 weakSet，不支持任何一种遍历方法（keys()、values()、entries()、forEach()），也没有 size 属性


## Example
1. 过滤字谜   
  + 字谜是指具有相同的字母，但是不一样排序的一些单词
    例如：nap - pan
         ear - are - era
         cheaters - hectares - teachers
  + 题目要求将一个数组中的字谜去掉，仅保留一个，例如：
    ```javascript
    let arr = ['nap', 'teachers', "cheaters", "PAN", "ear", "era", "hectares"]

    aclean(arr)   // 'nap', 'teachers', 'ear' 或 'PAN', 'cheaters', 'era'（反正每种字谜只保留一个）
    ```
  **解题思路**   
  + 由于相同的字谜中具有相同的字母，只不过顺序不同，那么不妨可以将这些单词的每个字母进行排序，相同的字谜排序出来的结果肯定是一样的   
  + 再用 object 或者 map 进行存储，排序后的新单词作为 key，则可以始终存储一个字谜中的单词（实际上只能得到最后一个）
  ```javascript
  function aclean(arr) {
    let map = new Map()

    arr.forEach(item => {
      let sortItem = item.toLowerCase().splite('').sort().join('')
      map.set(sortItem, item)   
      // 也可以用 object，因为 key 就是字符串
      // obj[sortItem] = item
    })

    return Array.from(map.values())   // 只要value即可
    // return Object.values(obj)
  }
  ```
