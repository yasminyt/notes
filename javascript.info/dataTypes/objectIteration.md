## Object.keys, values, entries
+ **`Object.keys(obj)`**：返回对象的所有 key 构成的数组
+ **`Object.values(obj)`**：返回对象的所有 value 构成的数组
+ **`Ojbect.entries(obj)`**：返回一个二维数组，其中的元素为 key 和 value 构成的一维数组
+ Object 中的以上方法与 Map 中对应的方法相比，有一些不同：

  | | **Map** | **Object** |
  | -------- | ---------- | ---------- |
  | Call syntax | map.keys() | Object.keys(obj)（而不是 obj.keys()） |
  | Returns | iterable | Array |
  
  + 调用方式不同：是 `Object.keys(obj)`，而不是 `obj.keys()`（这是为了增强对象的*灵活性*，因为可以给对象自定义 keys() 方法或是 values()，那么仍然可以通过 Object.keys 来获得对象的所有 key
  + Map 中的 keys()、values()、entries() 返回的是一个迭代器，即需要借助 `for...of`，而 Object 的直接返回的就是一个数组


### Object.keys/values/entries 忽略symbol属性
+ 和 `for...in` 类似，这些方法会忽略 symbol 类型的属性
+ 当然也可以有方法获取这类属性：
  + **`Object.getOwnPropertySymbols(obj)`**：返回对象中的所有 symbol 类型的 key 构成的数组
  + **`Reflect.ownKeys(obj)`**：返回所有的 key