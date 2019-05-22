# Scheduling: `setTimeout` & `setInterval`
+ 一个函数有的时候并不是想立即执行，而是过一段时间之后，这被称作是「计划调用」
+ 目前有两种方法可以实现：
  + `setTimeout`：将函数的执行推迟到一段时间之后再执行
  + `setInterval`：让函数隔一定时间周期性执行
+ 这两种方法并不存在于 JS 的规范中。但是大多数运行环境都有内置的调度器，而且也提供了这两种方法的实现，目前来说，所有浏览器，包括 Node.js 都支持这两种方法

## setTimeout
+ `let timerId = setTimeout(func|code, delay[, arg1, arg2, ...])`
  + `func|code`：想要执行的函数或代码字符串（一般传入的都是函数，由于历史原因，代码字符串也支持，但是不建议使用）
    - [x] 「这里只需要提供的是函数的引用，所以不能加上小括号，不然函数就会被执行（当然如果是对返回的闭包进行延迟执行，那就另当别论）」
  + `delay`：执行前的延时，以**毫秒**为单位「1000ms = 1s」
  + `arg1, arg2, ...`：要传入被执行函数 `func`（或代码字符串）的参数列表（IE9 以下不支持）
    - [x] 「所以这里需要注意当 `func` 需要传参时，只能通过 `arg1, arg2, ...`，而不能直接写在函数调用中，不然函数就会被立即执行」    

    ```javascript
    function sayHi(phrase, who) {
      alert(`${phrase}, ${who}`)
    }

    setTimeout(sayHi, 1000, 'Hello', 'John')  // 延迟1s后执行，输出 Hello, John
    ```
  + 如果第一个参数写入的是 字符串，js 会自动为其创建一个函数（相当于执行了 `new Function(str)`）
    ```javascript
    // 所以上述代码也可以简写成以下形式
    setTimeout("alert('Hello')", 1000)
    // 但是并不推荐，所以还是写成函数格式是最好的
    setTimeout(() => alert('Hello'), 1000)
    ```

### clearTimeout 取消调度
+ `setTimeout` 在调用时会返回一个「定时器 id」，那么可以根据这个 id 通过 `clearTimeout` 来取消调度
  ```javascript
  let timerId = setTimeout(...)
  clearTimeout(timerId)
  ```
+ 当取消掉调度时，`timerId` 会仍然保留着其值，而不会因为取消掉了调度任务而变为 `null`
  ```javascript
  let timerId = setTimeout(() => alert('nerver happens'), 1000)
  clearTimeout(timerId)
  alert(timerId)  // 还是原来的值，并没有因为调度被取消了而变成 null
  ```

## setInterval
+ `let timerId = setInterval(func|code, delay[, arg1, arg2])`
  + `setInterval` 和 `setTimeout` 的用法是相同的，所有参数的意义也是相同的
+ `setInterval` 是每间隔一定时间「周期性」执行函数，而 `setTimeout` 只执行一次
+ `clearInterval(timerId)` 阻止后续调用
```javascript
// 每 2s 执行一次
let timerId = setInterval(() => alert('tick'), 2000)
// 5s 后停止
setTimeout(() => { clearInterval(timerId); alert('stop') }, 5000)
```

### 弹框会让 Chrome/Opera/Safari 内的时钟停止
+ 在许多浏览器中，如 IE 和 Firefox 在显示 alert/confirm/prompt 时，内部的定时器仍旧会继续计时，但是在 Chrome、Opera 和 Safari 中，内部的定时器会暂停/冻结
+ 所以在执行上例代码时，如果在一定时间内（2s 内）没有关掉 alert 弹窗，那么在关闭弹窗后
  + Firefox/IE 会立即显示下一个 alert 弹窗（因为距离上一次执行超过了 2s）
  + 而 Chrome/Opera/Safari 则需要再等待 2s 以上的时间才会再显示（因为在 alert 弹窗期间，定时器并没有计时，而是关闭之后才开始计时）

## 递归 setTimeout
+ 周期性调度有两种方式：
  + 使用 `setInterval`
  + 递归调用 `setTimeout`
+ 递归调用 `setTimeout` 要比使用 `setInterval` 要灵活，采用这种方式可以根据当前执行结果来安排下一次调用
  + 例如，需要每间隔 5s 向服务器请求数据，如果服务器过载了，那么就要降低请求频率，比如将间隔增加到 10、20、40s 等（见以下的伪代码）
    ```javascript
    let delay = 5000
    let timerId = setTimeout(function request() {
      // ... 发送请求
      if (request failed due to server overload) {
        // 将下一次执行的间隔调整为当前的 2倍
        delay *= 2
      }
      timerId = setTimeout(request, delay)
    }, delay)
    ```
  + 再如，当不时有一些占用CPU的任务，可以通过衡量执行时间来安排下次调用是应该提前还是推迟
+ **递归调用 `setTimeout` 能保证每次执行间隔的延时都是准确的，`setInterval` 却不能够**
  + 比较下列两段代码，分别用了 递归调用 `setTimeout` 和 `setInterval`
    ```javascript
    // setInterval
    let i = 1
    setInterval(() => {
      func(i)
    }, 100)

    // 递归调用 setTimeout
    setTimeout(function run() {
      func(i)
      setTimeout(run, 100)
    }, 100)
    ```
  + 对于 `setInterval` 而言，内部的调度器会每间隔 100ms 执行一次 `func(i)`（如下图所示）
  ![fig1](https://zh.javascript.info/article/settimeout-setinterval/setinterval-interval.png)
    + 从图中可以看到：**使用 `setInterval` 时，`func` 函数的实际调用间隔要比代码给出的间隔时间要短**
    + 因为 `func` 的执行抵消了一部分间隔时间（也就是说，并不是函数执行完毕才开始计时，把函数的执行时间也一并计算了进去）
    + 那么如果函数的执行时间超过了设定的间隔时间（如上例，`func` 的执行时间超出了 100ms），这时 js 引擎会等待 `func` 执行完，然后向调度器询问是否时间已到，如果是，那么**立马**再执行一次。那么在极端情况下，如果函数每次执行时间都超过 `delay` 设定的时间，那么每次调用之间将毫无停顿
  + 而**递归调用 `setTimeout` 就能确保延时的固定**，因为「下一次调用是在前一次调用完成时再调度的」（如下图所示）
  ![fig2](https://zh.javascript.info/article/settimeout-setinterval/settimeout-interval.png)
+ ❗️垃圾回收
  + 当一个函数传入 setTimeout/setInterval 时，内存会为其创建一个引用，保存在调度器中，因此，即使这个函数没有被引用，也能防止 GC 将其回收
    ```javascript
    // 在调度器调用这个函数之前，这个函数将一直存在于内存中
    setTimeout(function() {...}, 100)
    ```
  + 需要注意的是，如果函数引用了外部变量（闭包），那么只要这个函数还存活着，外部变量也会随之存活，这样就可能会占用多于方法自身所需要的内存。所以，如果某个函数不需要再被调度，即使是个很小的函数，最好也将其取消

## setTimeout(..., 0)
+ `setTimeout(func, 0)`：0 延时调度用来安排在当前代码执行完时，需要尽快执行的函数
+ 这样的调度可以让 `func` 尽快执行，但是**只有在当前代码执行完后**，调度器才会对其进行调用
+ 换句话说，函数是在刚好当前代码执行完后执行，也就是**异步**
  + 例如，以下代码会先输出” Hello “，然后紧接着输出” World “
  ```javascript
  setTimeout(() => alert('World'), 0)
  alert('Hello')
  ```
  + 第一行代码“将调用安排到日程 0ms 处”，但是调度器只有在当前代码执行完毕时才会去“检查日程”，所以“ Hello ”先输出，然后才输出“ World ”
+ `setTimeout(..., 0)` 的用法示例：
  + 将耗费CPU的任务分割成多块，这样脚本运行不会进入“挂起”状态
  + 进程繁忙时也能让浏览器抽身做其他事情（例如绘制进度条）

### 分割CPU高占用的任务（详见👉 [click here](https://javascript.info/settimeout-setinterval#splitting-cpu-hungry-tasks)

### 给浏览器渲染的机会
+ 使用 `setTimeout(func, 0)` 分割任务还有好处，就是可以用来向用户反馈变化（如展示进度条等）
  + 「因为浏览器在所有脚本执行完后，才会开始 重绘（repainting）过程」
  + 所以，如果运行一个非常耗时的函数，即便在这个函数中改变了文档内容，除非这个函数执行完，那么变化是不会立刻反映到页面上的
  ```html
  <div id='progress'></div>

  <script>
    const progress = document.querySelector('#progress')
    function count() {
      for (let i = 0; i < 1e6; i++)
        progress.innerHTML = i
    }
    count()
  </script>
  ```
  + 运行上述代码时可以看到，`i` 值只会在整个计数过程完成后才显示
  + 接下来使用 `setTimeout` 对任务进行分割，这样就能在每一轮运行的间隙观察到变化了，这对用户来说是友好的
  ```html
  <script>
    let i = 0
    function count() {
      do {
        progress.innerHTML = i;
        i++
      } while (i % 1e3 != 0)   
      if (i < 1e9)
        setTimeout(count, 0)  // 执行完一轮时，则会重绘一次
    }
    count()
  </script>
  ```

## Note
+ 所有的调度方法都不能保证延时的准确性，所以在调度代码中，不可完全依赖
+ 浏览器内部的定时器会因各种原因而出现降速情况，如：
  + CPU过载
  + 浏览器页签切换到了后台模式
  + 笔记本电脑用的是电池供电（适用电池会以降低性能为代价提升续航）
