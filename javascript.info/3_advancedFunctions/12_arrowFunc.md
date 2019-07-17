# 深入箭头函数

## 箭头函数没有 this
+ 在前面 object 的内容中，也提及到了箭头函数没有 this，如果在箭头函数中使用 this 作为引用，那么 this 将会指向该箭头函数以外的正常函数被调用时的对象（如果没有，在非严格模式下，则是 window 对象）
+ 因此在对象方法中，如果需要在方法的函数中访问对象的某些属性或方法，那么使用箭头函数是最好的，因为 `this` 会指向该对象，而使用 `function` 声明的函数则做不到
  ```javascript
  let group = {
    title: 'our group',
    students: ['John', 'Pete', 'Alice'],

    showList() {    // 这里默认为 function 的简写形式
      this.students.forEach(student => alert(`${this.title}: ${student}`))
    }
  }

  group.showList()    
  ```
  + 由于是调用了 `group.showList()`，所以 `this.students` 中 `this = group`
  + 因为 `forEach()` 中使用了箭头函数，所以 `this.title` 中，`this` 指向了该箭头函数以外的 `function` 声明的函数被调用时的对象，即 `group`
  + 如果在 `forEach()` 中不是使用箭头函数，而是使用 `function` 声明的匿名函数，那么将会出错
    ```javascript
    let group = {
      title: 'our group',
      students: ['John', 'Pete', 'Alice'],

      showList() {    
        this.students.forEach(function(student) { 
          alert(`${this.title}: ${student}`) 
        })
      }
    }

    group.showList()   
    ```
  + 因为在普通函数中，`this` 一般指向了全局 `Window` 对象
+ 还有需要注意的是：因为箭头函数中没有 `this`，所以对象的方法不要用箭头函数来写，不然是无法读取外部对象
  ```javascript
  // 如上例中的 group.showList 如果用箭头函数来写，则报错
  let group = {
      title: 'our group',
      students: ['John', 'Pete', 'Alice'],

      showList: () => {    
        this.students.forEach(function(student) { 
          alert(`${this.title}: ${student}`) 
        })
      }
    }

    group.showList()    // Uncaught TypeError: Cannot read property 'forEach' of undefined
  ```

## 箭头函数没有 arguments 变量

## 箭头函数也不能使用 new

+ 所以总的来说，箭头函数适用于没有自己的“上下文”的短代码片段