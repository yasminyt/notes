# 回调
+ 在 JS 中，很多操作都是异步的
+ 换句话说，当某段代码中存在着等待操作时，js 引擎实际上并不会完全等待，而是会继续往下执行其它代码，这里称之为「同步」。如果此时在后面的代码中需要对等待后的结果进行处理，那么是无效的，因为不知道什么时候才完成，所以需要通过回调来监听这个完成的动作，然后再执行该处理，即「异步」
+ 例如在下例中，则是通过回调函数来完成异步操作：
  ```javascript
  function loadScript(src, callback) {
    let script = document.createElement('script')
    script.src = src
    // 通过 onload 事件来监听脚本是否已加载完毕
    script.onload = () => callback(script)
    
    document.head.append(script)
  }

  loadScript('https://cdnjs.cloudflare.com/ajax/libs/lodash.js/3.2.0/lodash.js', script => {
    alert(`Cool, the ${script.src} is loaded`)
    alert(_)   // _ 是 lodash 声明的方法，由于已经加载完毕，所以访问是有效的
  })
  ```

## 回调嵌套回调
+ 对于一些有顺序要求的异步操作来说，如果使用回调函数来进行处理，那么就需要将其余的回调函数嵌套在上一个回调函数之中
  ```javascript
  loadScript('/my/script.js', function(script) {
    alert('loaded')

    loadScript('/my/script2.js', function(script) {
      alert('also loaded')
    })
  })
  ```
+ 当这样的异步操作很多时，就会嵌套得很深：
  ```javascript
  loadScript('/my/script.js', function(script) {
    loadScript('/my/script2.js', function(script) {
      loadScript('/my/script3.js', function(script) {
        // ... maybe more
      })
    })
  })
  ```
+ 所以对于这样的情况，回调函数显然是不能满足需求的（虽然可以将代码进行拆分，但是明显还是非常冗余的）