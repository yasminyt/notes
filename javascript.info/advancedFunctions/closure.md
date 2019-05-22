# 闭包

### A couple of questions
+ 要理解闭包，先从以下两个简单的场景开始考虑，从而可以加深对闭包的理解，以及对闭包更复杂的使用
1. 函数 `sayHi` 使用了一个外部变量，但是在函数声明之后，这个外部变量的值发生了改变，那么函数在引用这个外部变量时，得到的是函数声明时变量的值，还是变量改变后的值？
  ```javascript
  let name = 'John'

  function sayHi() {
    alert(`Hi, ${name}`)
  }

  name = 'Pete'

  sayHi()   // 'Pete'
  ```
  + 在上例中，不过是使用函数声明，还是函数表达，都是一样的
2. 函数 `makeWorker` 返回了一个匿名函数。然后这个新的函数将会在任意地方进行调用，那么在该函数内部的变量访问的是它被创建时的同名变量，还是函数外部的全局变量，或者二者？
  ```javascript
  function makeWorker() {
    let name = 'Pete'

    return function () {
      alert(name)
    }
  }

  let name = 'John'
  let work = makeWorker()

  work() // 'Pete'
  ```

## Lexical Environment（词法环境）
+ 在 js 中，每个运行函数、代码块 {...} 和 脚本，作为一个整体都有一个**内部（隐藏）关联对象**，称为**词法环境**
+ 这个词法环境对象包括了以下两个部分：
  + 环境记录：一个存储了所有的变量作为其属性的对象（当然还包括别的一些信息，例如 this 的值）
  + 对外部词法环境的引用，用来与外部代码建立联系

### 变量定义
+ **“变量”只是特殊内部对象 Environment Record 的一个属性。「获取或更改变量的值」意味着「获取或更改该对象的属性值」**
+ 例如，在以下的简单代码中，只存在着一个词法环境
  ![fig1](https://zh.javascript.info/article/closure/lexical-environment-global.png)
  + 这是一个所谓的全局词法环境，与整个脚本相关联
  + 在上图中，方框表示了环境记录（变量存储）；箭头表示了外部引用，由于全局词法环境没有其它的外部引用，因此指向了 `null`
+ 当改变变量的值时，词法环境中所做的改变如下图所示：
  ![fig2](https://zh.javascript.info/article/closure/lexical-environment-global-2.png)
  + 右侧的矩形框演示了执行期间全局词法环境如何变化：
    1）当脚本开始执行时，词法环境是空的
    2）当 `let phrase` 定义出现，并且没有给变量赋值时，存储变量，并且默认赋值为 `undefined`
    3）给 `phrase` 赋值时，则改变其在词法环境中的值
    4）当重新给 `phrase` 赋值时，同样直接改变其在词法环境中的值
+ 所以总的来说：*变量只是一个特殊的内部对象的属性，并且与当前执行的代码块/函数/脚本相关联，改变变量的值，即改变该对象属性的值*
    
### 函数声明
+ 对于函数声明来说，它们的完全初始化并不是当执行到达时，而是更早，**当一个词法环境创建时，函数声明即被初始化**（注意这里是函数声明，而不是函数表达，前面在介绍二者时也有提及）
+ 所以，在一个脚本中，如果有函数声明，则当执行代码开始时，词法环境并不是空的，它会存放了声明的函数（与变量定义中的图进行对比）
  ![fig1](https://zh.javascript.info/article/closure/lexical-environment-global-3.png)

### 内部和外部词法环境
+ 那么函数在执行过程中，是如何访问全局变量的
+ 首先，*当执行一个函数时，一个新的函数的词法环境会被自动创建*，（这对所有函数都是适用的），这个函数的词法环境会存储函数中定义的局部变量和调用的参数
  + 例如，当执行函数调用时，有以下的结构：

  ![fig1](https://zh.javascript.info/article/closure/lexical-environment-simple.png)

  + 所以，在函数调用过程中，有两个词法环境：一个内部的（函数调用过程中自动产生的）、一个外部的（脚本执行时创建的全局的）：
    + 内部的词法环境会与函数 `say` 所在的全局外部词法环境相关联（内部词法环境中存储了定义在 `say` 中的变量，这里是 `name`，由于调用了 `say('John')`，所以给 `name` 赋值为 'John'）
    + 外部的词法环境是一个全局的词法环境（有全局变量和声明的函数）
+ **当函数内访问变量时，会先从自身内部的词法环境中查找，然后再到外一层，也就是说从内到外进行查找（因为有可能存在函数的多层嵌套声明），直到全局的词法环境**
  + 如果实在找不到，在严格模式下，会报错；在兼容模式下，会自动创建一个全局变量，以实现向后兼容
  + 上图的例子的查找过程为：
    + 在 `alert` 中想要访问 `phrase` 的值时，由于函数的词法环境中并没有该变量的值，所以顺着引用访问上一层词法环境（这里直接到了全局词法环境），在全局词法环境中查找到了 `phrase`
    + 再继续访问 `name` 的值时，由于函数的词法环境中已经存储了其值，所以直接可以得到

  ![fig2](https://zh.javascript.info/article/closure/lexical-environment-simple-lookup.png)

+ 所以在开篇所述的第一个场景中，`sayHi()` 的结果为 'Pete'

#### One call-one Lexical Environment
+ 每次函数执行时，都会生成一个新的词法环境
+ 如果一个函数被多次执行，那么**每个调用都将拥有自己的词法环境**，其中包含特定于该运行的局部变量和参数

#### 词法环境是一个特殊的对象
+ 词法环境是一个特殊的对象，无法在代码中获取，也不能直接操作它
+ 由 js 引擎对其进行优化，以及对内存进行管理

## 嵌套函数
+ 嵌套函数是指在函数中又创建并调用了新函数
  ```javascript
  function makeCounter() {
    let count = 0

    return function() {
      return count++
    }
  }

  let counter = makeCounter()
  alert(counter())  // 0
  alert(counter())  // 1
  alert(counter())  // 2
  ```
  + 如上示例，尽管返回的匿名函数被新赋值给了 `counter` 变量，但是由于该匿名函数与 `makeCounter()` 之间具有关联，所以在执行 `counter()` 时，仍然是会从内到外的词法环境中查找 `count`
+ 除了嵌套函数以外，定义在函数内部的局部变量怎么样都不能被该函数以外的地方访问到
+ 对于上述代码来说，每执行一次 `makeCounter()`，都会创建新的函数词法环境，所以 `count` 都会初始化为 0

### 嵌套函数的词法环境
+ 以上述代码为例，说明嵌套函数的词法环境，以解释清楚闭包中是如何对外层的参数和变量进行访问
+ 主要是注意现在多了一个 `[[Environment]]` 的属性
1. 当脚本开始执行时，只存在一个全局的词法环境（如下图所示）

  ![fig1](https://javascript.info/article/closure/lexenv-nested-makecounter-1.png)

  + 在一开始的时刻，全局词法环境中只有 `makeCounter` 函数，因为这是一个函数声明，所以一开始会先将函数声明放到全局词法环境中
  + **所有的函数在一“出生”的时候，就会自动获得一个隐藏的属性 `[[Environment]]`，该属性用于指向其创建的词法环境（也就是该函数在全局词法环境中的位置）**，所以函数就可以知道它是在哪里创建的
  + 从上图中可以看到，`makeCounter` 在全局词法环境中被创建，所以 `[[Environment]]` 保持了对它的引用
  + 换句话说，一个函数被“标记”，并引用它诞生时的词法环境；而 `[[Environment]]` 是具有该引用的隐藏函数属性

2. 当脚本继续往下执行时，一个新的全局变量 `counter` 被声明，并且其值为调用 `makeCounter()` 的结果（下图所示为执行 `makeCounter()` 第一行代码时的词法环境）

  ![fig2](https://javascript.info/article/closure/lexenv-nested-makecounter-2.png)

  + 在调用 `makeCounter()` 的时候，函数的词法环境被创建，用于存储它的变量和参数
  + 对于所有的词法环境，它存储了两个内容：
    + 具有局部变量的环境记录。在上述代码中，`count` 是唯一的局部变量，被存储在了 `makeCounter()` 的词法环境中（当执行 `let count` 这一行代码时即被存储）
    + 外部词法环境的引用，它被设置为函数的 `[[Environment]]` 属性，这里 `makeCounter` 的 `[[Environment]]` 引用了全局词法环境（就是可以理解为，在第1点中 `[[Environment]]` 的引用即为图中的 `outer` 引用）
  + 所以现在有了两个词法环境：1）全局环境；2）`makeCounter` 的词法环境，它拥有指向全局环境的引用

3. 在 `makeCounter()` 的执行中，创建了一个小的嵌套函数
  + 不管是使用函数声明或是函数表达式创建的函数，所有的函数都有 `[[Environment]]` 属性，该属性**引用着函数被创建时所在的词法环境**，所以这个新的嵌套函数也拥有这个属性
  + 新的嵌套函数的是在 `makeCounter()` 中创建的，所以它的 `[[Environment]]` 指向了 `makeCounter` 的词法环境（如下图所示）

  ![fig3](https://zh.javascript.info/article/closure/lexenv-nested-makecounter-3.png)

  + 这里函数并没有被立即调用，匿名函数内的代码也还没有执行，这里返回这个函数

4. 随着执行的进行，`makeCounter()` 调用完成（`return` 语句），并且将结果（该嵌套函数）赋值给全局变量 `counter`

  ![fig4](https://zh.javascript.info/article/closure/lexenv-nested-makecounter-4.png)

5. 当 `counter()` 执行时，同样会创建一个词法环境（「空」的，因为没有任何的局部变量），但是 `counter` 同时拥有了 3 中的 `[[Environment]]` 属性，所以它可以通过 `[[Environment]]` 访问前面创建的 `makeCounter()` 函数的变量（即通过外部引用访问 `makeCounter()` 函数的词法环境）

  ![fig5](https://zh.javascript.info/article/closure/lexenv-nested-makecounter-5.png)

  + **如果要访问一个变量，首先会搜索自身的词法环境（空），然后是前面创建的 `makeCounter()` 函数的词法环境，最后才是全局环境**
  + 所以当搜索 `count` 时，它会在最近的外部词法环境 `makeCounter()` 的变量中找到它
  + 需要注意这里的内存管理工作机制：虽然 `makeCounter()` 的执行已经结束，但是它的词法环境仍保存在内存中，因为这里仍然有一个嵌套函数的 `[[Environment]]` 在引用着它（3中所述）
  + 通常，只要有一个函数会用到该词法环境对象，它就不会被清理，并且只有没有（函数）会用到时，才会被清除

6. `counter()` 函数不仅会返回 `count` 的值，也会增加它。注意修改是「就地」完成的，也就是说是在找到 `count` 值的地方完成的修改

  ![fig6](https://zh.javascript.info/article/closure/lexenv-nested-makecounter-6.png)

  + 因此，在这一步中只有一处不同：即改变 `count` 的值。接下来的调用同理

+ 所以在开篇的第2个场景中，其对应的词法环境如下图所示：
  ![fig7](https://zh.javascript.info/article/closure/lexenv-nested-work.png)

# :exclamation: 闭包 :exclamation:
- [x] 函数能够保存其外部的变量并且能够访问它们称之为**闭包**
- [x] js 中函数都是天生的闭包（除了构造函数）
- [x] 也就是说，它们会通过隐藏的 `[[Environment]]` 属性记住它们被创建的位置，所以它们都可以访问外部变量
- [x] 在面试时，前端通常会被问到「什么是闭包」，应该回答闭包的定义，并且解释 js 中所有函数都是闭包，以及可能的关于 `[[Environment]]` 属性和词法环境原理的技术细节

## 代码块、循环、IIFE
+ 上述内容都是围绕函数说明的，实际上一个词法环境存在于任何一个代码块中 `{...}`
+ 当执行到一个代码块时，就会对应创建一个词法环境，并且包含了块内的变量

### if, for, while
+ 例如在以下 if 语句的代码块中，创建的词法环境包含了块内定义的变量 `user`
  ![fig1](https://javascript.info/article/closure/lexenv-if.png)

  + 当代码执行到 if 代码块时，则对应创建一个新的独属于 if 的词法环境
  + 由于存在了指向外部词法环境的引用，所以可以访问到 `phrase` 变量
  + 但是在 if 内的所有变量和函数表达式都无法被外部访问，所以 `alert(user)` 报错
+ 在循环的代码块内，也创建了其对应的词法环境，在 `for(let i = 0; ...)` 循环中，小括号内定义的变量也属于该代码块的词法环境内，即局部变量 `i` 也在 `for` 循环的代码块所创建的词法环境内
+ **需要注意的是，对于循环来说，每一次循环都会创建「新的词法环境」**

### 代码块
+ 可以通过使用花括号 `{...}` 来创建一个隔离的代码块，从而其内的变量被视为「局部变量」，则可以有效的规避下述所说的情况
  + 例如，在Web浏览器中，同一个应用程序的所有脚本（「type = 'module'」除外）共享相同的全局区域。因此，如果在一个脚本中创建一个全局变量，那么在其它脚本也可以使用它
  + 因此，在同一个应用程序中，有可能会出现全局变量被污染的情况
```javascript
{
  // 在该自定义的代码块内声明的变量、函数等，都不会被外部访问到
  let msg = 'Hello'
  alert(msg)  // Hello
}

alert(msg)  // Error: msg is not defined
```
+ 由于代码块拥有自己的词法环境，所以全局的词法环境无法访问

### IIFE
+ IIFE（immediately-invoked function expressions）—— 立即调用的函数表达式
  ```javascript
  (function () {
    let msg = 'Hello'
    alert(msg)
  })()
  ```
+ 在 IIFE 中，函数表达式被一对小括号括起来 `(function () {...})`；这是因为在主代码流中，js 一遇到「function」，就将其认为是函数声明 「`function funcName() {...}`」，但是函数声明需要一个函数名，所以这样的代码会被认为是错误的
+ 即便是加上了函数名，但是并不允许立即执行该函数
  ```javascript
  // syntax error because of parentheses below
  function go() {

  }();  // <-- can't call Function Declaration immediately
  ```
+ 因此，围绕函数的小括号是向 js 显示：函数是在「另一个表达式的上下文中创建」的，因此它是一个函数表达式，它不需要名称，可以立即调用

## 垃圾回收
+ 通常，在一个函数执行完成时，对应的词法环境会被清理和删除
+ 对于嵌套函数，如果内嵌的函数仍然被保存在变量中，那么外层函数的词法环境不会被清理，因为内嵌函数的 `[[environment]]` 属性仍然指向着外层函数的词法环境，也就是还存在着引用
  ```javascript
  function f() {
    let value = 123

    function inner() { alert(value) }

    return g
  }

  let outer = f()   // 由于嵌套函数 inner() 现在被 outer 变量访问，所以会仍然保留着对 f 的引用，那么 f() 不会被清理
  ```
  + 在上述代码中，如果 `f()` 被多次调用，则会存在多个词法环境
+ 当一个词法环境对象不可达时，则会被销毁（和普通的对象一样）。换句话说，只有至少一个嵌套函数指向它时它才会存在
  ```javascript
  // 在上述代码中添加以下代码
  g = null
  ```
  + 由于现在嵌套函数 `innter()` 已经不可达，所以它的词法环境先被清理，由于不再存在外部引用，所以 `f()` 的词法环境也被清理


## Example
+ 在数组的迭代函数中，自定义参数，实现更灵活的操作。以 `arr.filter()` 为例，如何通过闭包实现更灵活的筛选，由于 `filter()` 中需要一个函数作为筛选函数 `function (currentValue, index, arr)`，通过返回 true/false 来决定是否保留值
  ```javascript
  // 筛选数组中位于给定区间的元素
  function inBetween(a, b) {
    return function (x) {
      return x >= a && x <= b
    }
  }

  let arr = [1, 2, 3, 4, 5, 6, 7];
  arr.filter(inBetween(3, 6))   // 3, 4, 5, 6
  ```
  + 在上例中，当调用 `inBetween()` 时，直接返回了一个匿名函数，该匿名函数才是 `arr.filter()` 中的筛选函数，所以 `x` 可以直接获得数组的当前元素

+ 考虑循环中的闭包问题
  + 在以下代码中，`army` 中的每个元素执行的结果都为 10，并不符合预期的结果，即每个 army 都应该输出自己的标号 0、1、2、3、……、10
    ```javascript
    function makeArmy() {
      let shooters = []

      let i = 0
      while (i < 10) {
        let shoot = function () {
          alert(i)
        }
        shooters.push(shoot)
        i++
      }
      return shooters
    }

    let army = makeArmy()

    army[0]() // 10
    army[5]() // 10
    // 所有元素执行的结果都为 10
    ```
  + 输出结果之所以都为 10，是因为它们都访问了外层的变量 i，而 i 经过循环之后最后的结果为 10
  + 可以改变 i 所在的词法环境的范围，即缩小到与 `shoot` 在同一个词法环境中（每一次循环都会创建新的词法环境，所以每一次 i 的值都可以保留到当前的词法环境中）
    ```javascript
    // solution1
    function makeArmy() {
      let shooters = []

      for (let i = 0; i < 10; i++) {    // 修改为使用 for 循环
        let shoot = function () {
          alert(i)
        }
        shooters.push(shoot)
      }
      return shooters
    }

    // solution2
    function makeArmy() {
      let shooters = []

      let i  = 0
      while (i < 10) {
        let j = i       // 添加一个变量 j，j 与 shoot 在同一个词法环境中
        let shoot = function () {
          alert(j)
        }
        shooters.push(shoot)
      }
      return shooters
    }
    ```
  + 「solution1」的简单的词法环境可以由下图所示
  ![fig1](https://javascript.info/task/make-army/lexenv-makearmy.png)