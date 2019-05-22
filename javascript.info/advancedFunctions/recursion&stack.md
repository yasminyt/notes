# 递归和堆栈
## 执行堆栈
+ 一个函数运行的信息被存储在它的**执行上下文**里
+ 执行上下文 是一个内部数据结构，它包含了一个函数执行时的细节：当前工作流在哪里、当前的变量、this 的值，以及其它一些内部细节
+ 每个函数调用都有与其相关联的执行上下文
+ 当一个函数有嵌套调用时，会发生下面的操作：
  + 当前函数被暂停
  + 与它关联的执行上下文被 **执行上下文堆栈** 的特殊数据结构保存
  + 执行嵌套调用
  + 嵌套调用结束后，之前的执行上下文从堆栈中恢复，外部函数从停止的地方继续执行
+ 由于上下文的保存、切换都会产生一定的内存消耗，因此递归相对于循环来说对内存的要求要更高一点
+ 递归能提供更简洁的代码，容易理解和维护，不是什么时候都需要优化，有的时候可能更需要好的代码，因此递归得到了广泛的使用

## 递归遍历
+ 递归另一个重要应用就是递归遍历
+ 例如，有以下对象结构用来描述某个公司的职员结构
  ```javascript
  let company = {
    sales: [{name: 'John', salary: 1000}, {name: 'Alice', salary: 600}],  // （1）
    development: {    // （2）
      sites: [{name: 'Peter', salary: 2000}, {name: 'Alex', salary: 1800}],
      internals: [{name: 'Jack', salary: 1300}]
    }
  }
  ```
+ 该结构也有可能会更复杂，例如： `sites` 属性的值有可能以后会继续划分为多个子属性；`development` 属性中可能不止这几个属性，还有可能会继续增加；`company` 这个对象中的属性的结构也不一定都相同；等等。所以如果当想要计算所有员工的总的薪酬时，使用 for 循环进行遍历是非常复杂的解决方案。但是通过递归，则会是非常简单的实现（其实与深拷贝的实现比较类似）
+ 分析：可以看到，当使用函数计算每一个部门的薪酬之和时，有两种可能情况：
  1）这个部门只是具有简单的一组人的数据（见代码（1））—— 则可以使用简单循环来计算薪酬总额
  2）具有 N 个子对象的部门（见代码（2））—— 可以通过 N 次递归调用来求每一个子部门的总额然后再进行合并
  + 可以看到：1）是递归的基础，是结束递归的关键；2）是递归的步骤，复杂的任务被划分为适用于更小部门的子任务，（即便它们以后还会添加更多的子任务，或是划分为更细的子任务）但是最终都会在 1）那里完成计算并结束
+ 代码实现：
  ```javascript
  function sumSalaries(department) {
    if (Array.isArray(department))
      return department.reduce((prev, curr) => prev + curr.salary, 0)   // 计算一个部门的薪酬，这里也是递归结束的条件
    else {
      let sum = 0
      for (let subdep of Object.values(department)) {
        sum += sumSalaries(subdep)  // 计算每个子部门的薪酬
      }
      return sum
    }
  }
  ```

## 递归结构
+ 递归数据结构是一种复制自身的结构
+ 在 web 中，可以从 HTML 和 XML 文档来看递归
+ 在HTML文档中，一个HTML标签可能包括一组：
  + 文本片段
  + HTML 注释
  + 其它 HTML标签（它有可能又包括文本片段、注释或其它标签等）
+ 链表也属于递归结构，当需要对大量数据进行增删操作时，它是优于数组的选择（对链表的介绍与实现见 lear.fcc 中的数据结构的笔记）

## Example
+ 反向输出链表中各项的值
  ```javascript
  let list = {
    value: 1,
    next: {
      value: 2,
      next: {
        value: 3,
        next: {
          value: 4,
          next: null
        }
      }
    }
  }
  ```
  + 使用递归
    ```javascript
    function rePrintList(list) {
      if (list.next) 
        rePrintList(list.next)  // 先到达链表的尾巴

      console.log(list.value)
    }
    ```
  + 使用循环
    ```javascript
    function rePrintList(list) {
      let tmp
      // 先遍历链表获得所有的值
      while(list) {         
        tmp.push(list.value)
        list = list.next
      }
      // 倒序输出所有的值
      for (let i = tmp.length - 1; i >= 0; i--)
        console.log(tmp[i])
    }
    ```
  + 使用递归要比使用循环，代码更加简洁
  + 实际上递归的实现与循环是类似的，因为每一次递归时，都要在执行上下文堆栈里去记录每一次嵌套调用里链表上的元素，当返回到链表的尾巴时，再依次输出