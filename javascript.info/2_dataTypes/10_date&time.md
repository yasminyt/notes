## 创建时间日期对象
+ **`new Date()`**
  + 无参数时，创建的日期对象将得到当前的日期和时间
+ **`new Date(milliseconds)`**
  + 提供以毫秒为单位的时间戳，则可以获得 标准时间（1970-1-1 UTC+0）（时间会根据代码执行时的当地时区来确定） + 时间戳 的时间
  ```javascript
  // 0 意味着直接获得基准时间 1970-1-1 UTC+0
  let Jan01_1970 = new Date(0) 

  // 为了获得 1970-1-2 UTC+0，则传递 24 小时的毫秒时间戳
  let Jan02_1970 = new Date(24 * 3600 * 1000)
  ```
+ **`new Date(datestring)`**
  + 传递时间序列的字符串，年月日之间可以用 空格、下划线'_'、反斜杠'/' 来进行间隔，也可以混合使用；日期和时间之间用空格隔开；当年月日之间用空格隔开时，如果要加上时间，而又只有时（没有分秒），必须要加上一个分号，以说明这是时间（见第二个代码实例）
  ```javascript
  new Date('2019-4-5')
  // Fri Apr 05 2019 00:00:00 GMT+0800 (中国标准时间)

  new Date('2019 4 5 20:')  // 必须给时加上分号，否则会被视为非法的
  // Tue Apr 23 2019 20:00:00 GMT+0800 (中国标准时间)
  ```
+ **`new Date(year, month, date, hours, minutes, seconds, ms)`**
  + `year` 和 `month` 这两个参数是必须的（否则就会被视为时间戳）
  + `year` 必须由 4 个数字组成
  + `month` 必须从 0（表示1月）开始，以 11（表示12月）结尾
  + `date` 如果缺省，则默认为 1
  + `hours/minutes/seconds/ms` 如果缺省，则默认为 0

## 获得时间日期的各部分内容
+ 对创建的时间日期对象，可以通过以下方法来获得各部分的内容
+ **`getFullYear()`**：获得 年份（4位数字）
+ **`getMonth()`**：获得 月份（0 ～ 11）
+ **`getDate()`**：获得 日期（1 ～ 31）
+ **`getHours()`**、**`getMinutes()`**、**`getSeconds()`**、**`getMilliseconds()`**：分别获得 时、分、秒、毫秒
+ **`getDay()`**：获得 星期几，0（星期天）～ 6（星期六）
+ **`getTime()`**：获得 该日期时间对象 相对于 标准时间 的时间戳

### 重新设置时间日期对象的各部分内容
+ 可以对生成的时间日期对象重新设置其各部分内容
+ **`setFulllYear(year [, month, date])`**
+ **`setMonth(month [, date])`**
+ **`setDate(date)`**
+ **`setHours(hour [, min, sec, ms])`**
+ **`setMinutes(min [, sec, ms])`**
+ **`setSeconds(sec, [ms])`**
+ **`setMilliseconds(ms)`**
+ **`setTime(milliseconds)`**：实际上已经重新设置整个时间日期对象的内容，相当于执行了一次 **`new Date(milliseconds)`**，只不过是说如果变量是 const 声明的，则可以通过 `setTime(milliseconds)` 重新修改变量的内容

### 自动更正
+ Date 对象具有 *自动更正* 的功能，因此可以设置超出范围的值，它会自动调整
  ```javascript
  new Date(2019, 1, 30) 
  // Sat Mar 02 2019 00:00:00 GMT+0800 (中国标准时间)
  // 自动修正为 3月2号
  ```

## 时间日期 与 数字 之间的转换
+ 当需要把一个时间日期对象转换为数字时，可以直接在该对象前使用*一元运算符 '+'*，则会得到该时间对应的时间戳，等同于使用了 `date.getTime()`
  ```javascript
  let date = new Date()
  +date   // 1558076180426
  ```
+ 两个时间日期对象可以直接相减，结果为二者的时间戳之差，以 ms 做单位（相减时，会先将时间日期对象转换为时间戳，所以会带来一定的性能损耗）

## Date.now()
+ **`Date.now()`**：返回当前时间的时间戳
+ 该方法的好处是，当只想要获得当前时间的时间戳时，则可以不用通过生成一个 Date 对象，再调用 getTime() 方法，所以会更快，也不会增加内存的负担

## Date.parse()
+ **`Date.parse(str)`**：返回时间日期字符串所对应的时间戳
+ str 的形式为：**YYYY-MM-DDTHH:mm:ss.sssZ**
  + **YYYY-MM-DD**：年月日
  + **T**：分隔符
  + **HH:mm:ss.sss**：时分秒、毫秒
  + **Z**：可选的，表示时区，Z 表示 UTC+0
