## 字符访问
+ 对字符串中的字符进行访问，通常通过方括号 `"[]"`
+ 也可以通过 `str.charAt(pos)` 方法来访问，`pos` 为字符的在字符串中的位置，和使用方括号是一样的
  ```javascript
  let str = 'Hello'

  str.charAt(0)   // H
  ```
  + 该方法用得比较少，它的存在主要是因为历史原因
+ `"[]"` 与 `charAt` 方法唯一的区别是，*如果字符串为空，或者访问的位置超过了字符串的长度，前者返回的是 `undefined`，后者返回的是一个空串
  ```javascript
  str[100]  // undefined
  str.charAt(100) // '' (an empty string)
  ```
+ 也可以通过 `for...of` 来遍历字符串
  ```javascript
  for (let char of str)
    alert(char)     
  ```

## 取子串
+ `str.slice(start [, end])`： 取从 start 位置到 end 位置的子串（不包括 end 位置的字符）；如果没有 end 参数，则从 start 位置一直取到字符串尾
  + start 和 end 都可以为 负数，为负数时表示从后面开始取，从 -1 开始计数位置（没有 -0 来说）
    ```javascript
    let str = 'stringify'
    // 从后面倒数第4个位置开始取（g），取到倒数第一个位置（-1）的字符（y）的前一个字符（f）
    str.slice(-4, -1)   // gif
    ```
+ `str.substring(start [, end])`：和 `slice()` 类似，但是主要表达的是 start 和 end 之间的字符串（不包括 end），所以有两点不同：
  + *start 的值可以**大于** end 的值*（`slice()` 中如果 start 大于 end，则返回一个空串）
    ```javascript
    str.substring(2, 6)   // ring
    str.substring(6, 2)   // ring

    str.slice(6, 2)     // '' (an empty string)
    ```
  + *start 和 end 的值不能都是负数，如果其中一个是负数，则看作是 0，如果两个都是负数，则返回一个空串*
+ `str.substr(start [, length])`：取从 start 位置开始，length 长度的字符串；如果没有指定 length，则从 start 位置一直取到字符串尾（和 `slice()` 类似）
  + start 的值可以是负数，表示倒数第几个位置的字符开始取
    ```javascript
    str.substr(2, 4)  // ring
    str.substr(-4, 2) // gi
    ```
+ 几种方法的比较、总结
  + 以上三种方法都只是**取**子串，所以不会改变原本的字符串

  | method | selects | negatives |
  | ------ | ------- | --------- |
  | slice(start, end) | from start to end (not including end) | 允许负数 |
  | substring(start, end) | between start and end (not including end) | 将负数看作是0 |
  | substr(start, length) | from start get length characters | 允许 start 为负数 |