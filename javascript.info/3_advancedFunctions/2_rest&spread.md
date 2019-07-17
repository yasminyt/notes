# Rest parameters & spread operator

## Rest parameters `...`
+ 当函数的参数列表中参数的个数不固定时，可以使用 `...` 操作符，表示*将剩余的参数收集到一个数组中*
+ 使用 `...` 操作符声明的变量是一个**数组**，所以可以使用数组的方法进行处理
  ```javascript
  function sum(...nums) {   // nums 是一个由参数构成的数组
    return nums.reduce((prev, curr) => prev + curr, 0)
  }

  sum(1, 2)   // 3
  sum(1, 2, 3)  // 6
  sum(1, 2, 3, 4)   // 10
  ```
+ 可以将 `...` 操作符与一般的形参变量一起使用，但是 `...` 操作符必须放在最后，毕竟表示的是剩余的变量
  ```javascript
  function showName(first, last, ...rest) {
    console.log(first)        // 'Julius'
    console.log(last)         // 'Caesar'
    // rest = ['Consul', 'Marry']
    console.log(rest.length)  // 2 
  }

  showName('Julius', 'Caesar', 'Consul', 'Marry')
  ```

### `arguments` 变量
+ **`arguments`** 变量是一个类数组的特殊对象，也包括了所有的形参，并且可以按照每个参数在传递时定义的顺序像数组那样进行访问
  ```javascript
  function showName() {
    console.log(arguments.length)   // 4
    console.log(arguments[0])     // 'Julius'
    console.log(arguments[1])     // 'Caesar'
  }

  showName('Julius', 'Caesar', 'Consul', 'Marry')
  ```
+ 由于 `arguments` 变量并不是一个真正的数组，所以不能使用数组的方法，并且始终包含了所有的形参
+ **箭头函数中没有 `arguments` 变量**，如果箭头函数内嵌在一个使用 `function` 定义的函数中，则 `arguments` 访问的是这个外层函数的所有形参
  ```javascript
  function func() {
    let showArg = () => console.log(arguments.length)
    showArg()
  }

  func(1, 2)  // 2
  ```

## Spread operator
+ 在函数调用中，可以使用 `...` 操作符来获得可迭代的变量的元素构成的参数列表（`...` 操作符在函数定义中作为 rest parameters，在函数调用时作为 spread operater）
  ```javascript
  let arr = [1, 2, 3]
  Math.max(...arr)  // 3
  ```
+ 一次可以使用多个 `...`
  ```javascript
  let arr1 = [1, 2, 3],
      arr2 = [4, 5, 8]

  Math.max(...arr1, ...arr2)  // 8
  ```
+ `...` 可以和普通值一起使用
  + 函数调用
    ```javascript
    Math.max(-1, ...arr1, ...arr2, 10)  // 10
    ```
  + 新的数组
    ```javascript
    let newArr1 = [...arr1, ...arr2] // newArr1 = [1, 2, 3, 4, 5, 8]
    let newArr2 = [0, ...arr1, 10]  // newArr2 = [0, 1, 2, 3, 10]
    ```
+ 扩展运算符可以用在一切可迭代的数据类型上，如数组、字符串、Map、Set，相当于对它们使用了 `for...of`
+ 但是当扩展运算符不是使用在函数调用时，需使用在数组中，即会创建一个新的数组，其结果等同于 `Array.from`
  ```javascript
  let str = 'Hello'

  let arr = [...str]  // arr = ['H', 'e', 'l', 'l', 'o']
  // 等同于 arr = Array.from(str)
  ```
  + 但是 `Array.from` 更加通用，因为可以作用在对象上，而扩展运算符是不能用在对象上的
+ 扩展运算符不可以单独使用，也就是说不能单独写在赋值运算符后，会报错，如下写法是不正确的
  ```javascript
  let arr = [0]
  let zero = ...arr   // error!
  ```