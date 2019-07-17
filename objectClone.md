# Object 深拷贝&浅拷贝方式汇总

## 深拷贝
+ 主要是通过递归的方式来实现
  ```javascript
  function deepClone(obj) {
    let result = Array.isArray(obj) ? [] : {}

    if (obj && typeof obj == 'object') 
      for (let elem in obj)
        if (obj.hasOwnproperty(elem)) 
          if (obj[elem] && typeof obj[elem] == 'object')
            result[elem] = deepClone(obj[elem])
          else
            result[elem] = obj[elem]

    return result
  }
  ```
+ JSON 序列化和反序列化的方式
  ```javascript
  JSON.parse(JSON.stringify(obj))
  ```
  + 该方法简单，易实现
  + 但是 `JSON.stringify` 的局限比较多，例如对于属性值为 `undefined`、Map、Set 类型的属性、方法属性、Symbol 类型的属性都会忽略掉

## 浅拷贝
+ 通过属性标签和描述符的方式
  ```javascript
  let clone = Object.defineProperties({}, Object.getOwnPropertyDescriptors(obj))
  ```
  + `getOwnPropertyDescriptors` 可以获得包括 Symbol 类型、方法属性在内的所有属性
+ `Object.create` 方法
  ```javascript
  let clone = Object.create(Object.getPropertyOf(obj), Object.getOwnPropertyDescriptors(obj))
  ```
  + 该方法与上述方法类似，都可以对特殊的属性值进行浅拷贝
  + 但是上述的两种方法也没有将对应的原型关系进行拷贝
+ 遍历的方式
  ```javascript
  function clone(obj) {
    let clone = Array.isArray(obj) ? [] : {}

    for (let item in obj) 
      clone[item] = obj[item]

    return clone
  }
  ```
