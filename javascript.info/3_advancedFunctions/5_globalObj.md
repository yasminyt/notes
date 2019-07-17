# 全局对象
+ 全局对象提供的变量和方法可以在任何地方使用
+ 在浏览器中，全局对象为「window」，对 Node.js 来说，是「global」

## 浏览器：「window」对象
+ 由于历史原因，浏览器中的 `window` 对象被弄得有点乱   
+ 除了扮演全局对象的角色之外，它还提供“浏览器窗口”功能：可以使用 `window` 来访问特定于浏览器窗口的属性和方法
  ```javascript
  window.innerHeight    // 获取浏览器的窗口高度

  window.open('http://google.com')  // 打开一个新的浏览器窗口
  ```
+ 使用 `var` 声明的变量和函数，会自动成为 `window` 的属性
  ```javascript
  var x = 5
  alert(window.x)    // 5（变量 x 成为 window 的一个属性

  window.x = 0
  alert(x)    // 0
  ```
  + 但是使用 `let` / `const` 不会有这样的情况
+ 所有脚本共享相同的全局作用域，因此在某一个 `<script>` 中声明的变量在其他的脚本中可见
  ```html
  <script>
    var a = 1
    let b = 2
  </script>

  <script>
    alert(a)  // 1
    alert(b)  // 2
  </script>
  ```
+ 全局范围内，非严格模式下，`this` 的值是 `window`

### type = 'module'
+ 不同脚本（可能来自不同的源）之间的变量可以互相访问，会导致命名冲突：相同的变量名可能在不同脚本中被用于不同目的，因此这些变量名将相互冲突
+ 给 `<script>` 标签属性 `type = "module"`，那么这样的脚本被认为是个单独的“模块”，它有自己的顶级作用域（词法环境），不会干扰 window
  + 在一个模块中，`var x` 不会成为 window 的属性
    ```html
    <script type='module'>
      var x = 5
      alert(window.x)   // undefined
    </script>
    ```
  + 两个模块的变量彼此不可见
    ```html
    <script type='module'>
      let x = 5
    </script>

    <script type='module'>
      alert(window.x) // undefined
      alert(x)    // undefined
    </script>
    ```
  + 模块中 `this` 的值是 `undefined`
+ **使用 `<script type='module'>` 后，通过将顶级作用域与 window 分开的方式来修复语言的设计缺陷**
