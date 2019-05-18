## 性能比较
+ push/pop 方法执行快，而 shift/unshift 方法执行慢

+ `array.shift()` 在执行过程中，必须要做以下三件事：
  1. 移除 `index = 0` 的元素
  2. 将剩下的元素全部向左移动，并且将它们的索引从 1 重新编号为 0，从 2 重新编号为 1，依此类推
  3. 更新 `length` 属性
  
  **数组中的元素越多，移动它们的时间就越多，内存操作也就越多**
  + `unshift()` 的操作也类似，需要先将所有的元素向右移动一个位置，并且重新编号索引，将元素添加进去之后，再重新更新 length 的值
+ 而 pop/push 方法的效率高主要是因为不需要移动任何元素

## 循环数组
+ 有三种方式可以循环列举数组的所有元素
+ `for` 循环
+ `for...of` 循环，该方法会更加方便
  ```javascript
  let numbers = [1, 2, 3, 4]

  for (let number of numbers)
    alert(number)   // 1, 2, 3, 4
  ```
  + `for...of` 循环不需要通过当前元素的位置来访问元素，所以写法上更简洁
+ `for...in` 循环，由于数组就是对象，所以可以通过该方式来遍历元素
  ```javascript
  for (let i in numbers)
    alert(numbers[i])
  ```
+ 上述三种方法中，`for...in` 是最不提倡的
  + 因为该方法是针对 object 作的优化，而不是数组，所以遍历速度会慢了 10-100 倍
  + 另外该方法会将所有的属性都给遍历完，而不只是数字属性，例如如果添加了这样的属性 `numbers.test = 'apple'`，则方法会将 test 属性也给遍历出来，可是这并不是我们所需要的

## length 属性
+ 每当数组发生改变时，都会自动更新 length 属性
+ 准确来说，length 并不是简单计算数组中所有元素的个数，而是**最大的位置索引 + 1**
+ length 属性是可以被手动更改的，若是减小 length 的值，则是对数组作删除操作（即删除了超过指定长度后的元素）；若是增大 length 的值，则访问没有值的位置时，返回 undefined
  ```javascript
  let arr = [1, 2, 3, 4, 5]

  arr.length = 2  
  alert(arr)  // [1, 2]

  arr.length = 5
  alert(arr[3])   // undefined
  ```
+ 所以清空一个数组最简单的方式是 `arr.length = 0`（其实令 `arr = []` 不是更方便点，当然前提是不能是 const）

## new Array()
+ `new Array()` 方法可以用于创建一个新的数组对象，这种方式并不常用，通过方括号的方式是更普遍和经常的用法
+ `new Array()` 的参数有三种形式：
  + 没有参数，则表示创建一个空的数组对象
    ```javascript
    let arr = new Array()
    // 等同于 let arr = []
    ```
  + 将要添加的元素作为参数
    ```javascript
    let arr = new Array('apple', 'pear', 'etc')
    ```
  + 如果只有一个数字作为参数，则创建 length 的值为该数字的数组对象，得到一个空数组
    ```javascript
    let arr = new Array(2)    // 创建长度为 2 的数组
    // 得到的 arr 等同于 arr = [,,]

    alert(arr.length)   // 2
    alert(arr[0])       // undefined (在对每个元素进行访问时，都会是 undefined)
    ```
+ **`new`** 关键字可以被省略，在 ES5 标准中，明确指出就算省略了该关键字，也会是创建和初始化一个新的 Array 对象，所以 `Array()` 和 `new Array()` 的行为是一致的

## toString
+ 数组也可以有“加法”运算，即可以使用 + 
  ```javascript
  [] + 1       // "1"
  [1] + 1      // "11"
  [1, 2] + 1   // "1,21"
  ```
  + 从以上代码中可以看到，当数组使用 + 时，+ 作为连接符
  + 数组先会被转为字符串（各个元素用逗号相隔形成的字符串）
  + “+” 后面的内容也会被转成字符串，二者再拼接起来

## Demo
+ **在数组上下文进行调用**
  ```javascript
  let arr = ['a', 'b']

  arr.push(function () {
    alert(this)
  })

  arr[2]()    // 'a', 'b', function
  ```
  + 上述代码的执行结果见注释所示
  + `arr.push()` 添加了一个函数作为数组的对象
  + 所以 `arr[2]()` 对该函数进行调用，并且数组本身就是个对象，所以为典型的 `obj[method]()` 的方式，只是这里是个匿名函数，所以通过下标访问即可，那么 `this == arr`

+ **最大的子阵列**
 
  > 问题描述：`getMaxSubSum(arr)` 方法接受一个数组作为参数，要求找到具有最大项目总和的 arr 的连续子序列，并返回这个总和。
  > 例如：arr = [2, -1, 2, 3, -9] 中最大项目总和为 6，由 [2, -1, 2, 3] 该子串所得

  ```javascript
  // 该结果代码已经算是最快的解法了，O(n)级别就可以完成
  /** 思路是：
   * （1）使用 partialSum 变量来记录当前元素与前面元素相加的和，当和为负数时，则重置 partialSum 为 0（因为就算再加一个正数，总和也变小了，所以要归零）；
   * （2）使用 maxSum 变量记录当前最大的总和
   */

  function getMaxSubSum(arr) {
    let partialSum = 0, 
        maxSum = 0

    for (let num of arr) {
      partialSum += num
      maxSum = Math.max(partialSum, maxSum)
      if (partialSum < 0)
        partialSum = 0
    }

    return maxSum
  }
  ```