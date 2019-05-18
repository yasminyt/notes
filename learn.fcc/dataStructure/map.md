## Map
+ JS 的对象（object），本质上是键值对的集合（Hash结构），但是只能用字符串当作键，在使用过程中会有比较大的限制
+ 为了解决上述问题，ES6 添加了 Map 数据结构
+ Map 类似于 Object，也是键值对的集合，但是 键 的范围不限于字符串，各种类型的值（包括对象）都可以当作键
+ 也就是说，Object 结构提供了“字符串——值”的对应，Map 结构提供了“值——值”的对应，是一种更完善的 Hash 结构实现

## Map 构建
+ 通过 `new Map()` 构造函数创建 Map 数据结构
+ `Map()`接受一个数组作为参数，该数组的成员为一个个表示键值对的数组，也就是说是一个**二维数组**
  ```javascript
  const map = new Map([['name', 'Lisa'], ['title', 'Author']])

  //或创建一个空的 Map
  const emptyMap = new Map()
  ```
+ 如果对同一个键多次赋值，后面的值将**覆盖**前面的值（不存在重复的键）
+ 如果读取一个未知的键，则返回 `undefined`  
   *Note：只有对同一个对象的引用，Map 结构才将其视为同一个键*（如下代码所示）     

   ```javascript
   const map = new Map()
   map.set(['a'], 555)
   map.get(['a'])  // undefined
   ```    
   
   - 上面代码的 `set()` 和 `get()`，表面是针对同一个键，但实际上这是两个值，内存地址是不一样的，因此 `get()` 无法读取该值，返回 `undefined`    
   - 其实通过 `['a'] == ['a']  // false` 可以知道，对于一个应用类型的数据，其返回的是一个内存地址，所以对于每一个数组都会新分配一个内存区，所以 `map.get(['a'])` 的结果为 `undefined`    
+ Map 的键实际上是跟**内存地址**绑定的，只要内存地址不一样，就视为两个键（如上述所示）
  - 这就解决了同名属性碰撞（clash）的问题
  - 在扩展别人的库的时候，如果使用对象作为键名，就不用担心自己的属性与原作者的属性同名
+ 如果 Map 的键是一个简单类型的值（Number、String、Boolean），则只要两个值**严格相等**，Map 将其视为*一个值*，包括`0` 和 `-0`。另外，虽然 `NaN` 不严格相等于自身，但 Map 将其视为同一个键
  ```javascript
  let map = new Map()

  map.set(NaN, 123)
  map.get(NaN)  // 123

  map.set(-0, 123)
  map.get(+0)   // 123
  ```

## 属性和操作方法
+ `size`：返回成员总数
+ `set(key, value)`：设置 `key` 对应的键值，然后返回整个 Map 结构
  - 如果 `key` 已经有值，则键值会被更新，否则就新生成该键
  - 返回的是 Map 本身，所以也可以使用链式写法
+ `get(key)`：读取 `key` 对应的键值，如果找不到 `key`，则返回`undefined`
+ `has(key)`：返回一个*布尔值*，表示某个键是否在 Map 数据结构中
+ `delete(key)`：删除某个键，返回 `true`，如果删除失败，返回 `false`
+ `clear()`：清除所有成员，没有返回值

## 遍历方法
+ `keys()`：返回键名的遍历器
+ `values()`：返回键值的遍历器
+ `entries()`：返回所有成员的遍历器
  - Map 结构的默认遍历器接口，就是 `entries()`，所以以下两种写法是等同的
    ```javascript
    const map = new Map([['F', 'no'], ['T', 'yes']])

    for (let [key, value] of map.entries())
      console.log(key, value)

    //等同于使用 map.entries()
    for (let [key, value] of map) 
      console.log(key, value)
    ```
+ 同样可以使用扩展运算符（...）将 Map 结构转化为 数组
  ```javascript
  let map = new Map([[1, 'one'], [2, 'two'], [3, 'three']])

  [...map.keys()]   // [1, 2, 3]
  [...map.values()] // ['one', 'two', 'three']

  // 以下两种写法等同
  [...map.entries()] // [[1, 'one'], [2, 'two'], [3, 'three']]
  [...map]           // [[1, 'one'], [2, 'two'], [3, 'three']]
  ```