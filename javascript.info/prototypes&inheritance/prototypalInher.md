# 原型继承「prototypal inheritance」

## `[[Prototype]]`
+ js 中，对象具有特殊的隐藏属性 `[[Prototype]]`，其属性值为 `null` 或引用另一个对象。这个对象也被称作为「原型」，如下图所示：
  ![fig1](https://javascript.info/article/prototype-inheritance/object-prototype-empty.png)
+ 属性 `[[Prototype]]` 是内部隐藏的，但是可以通过别的方式来设置它
+ `__proto__` 可以定义一个对象的原型是什么，即指明 `[[Prototype]]` 的指向
  ```javascript
  let animal = {
    eats: true
  }
  let rabbit = {
    jumps: true
  }

  rabbit.__proto__ = animal
  ```
  + 在上述代码中，指明了 `rabbit` 的原型为 `animal`，如下图所示：
  ![fig2](https://javascript.info/article/prototype-inheritance/proto-animal-rabbit.png)
  + 那么可以说，「animal 是 rabbit 的原型」或者「rabbit 原型继承自 animal」
  + 如果在 `rabbit` 中访问某个属性时，如果该属性缺失，那么会顺着 `[[Prototype]]` 的指向查找 `animal` 中是否有该属性，如果有则返回，否则继续顺着 `animal` 的 `[[Prototype]]` 的指向继续查找，除非其值为 `null`
    ```javascript
    alert(rabbit.eats)    // true
    alert(rabbit.jumps)   // true
    ```
  + 如果在 `animal` 中有一个方法，那么也可以通过 `rabbit` 调用：
    ```javascript
    let animal = {
      eats: true,
      walk() {
        alert('Animal walk')
      }
    }
    let rabbit = {
      jumps: true,
      __proto__: animal
    }

    rabbit.walk()   // Animal walk
    ```
    + 方法自动从原型对象中获取：
      ![fig3](https://javascript.info/article/prototype-inheritance/proto-animal-rabbit-walk.png)
+ 原型链可以更长：
  ```javascript
  let animal = {
    eats: true,
    walk() {
      alert('Animal walk')
    }
  }
  let rabbit = {
    jumps: true,
    __proto__: animal
  }
  let longEar = {
    earLength: 10,
    __proto__: rabbit
  }

  longEar.walk()    // Animal walk
  longEar.jumps     // true
  ```
  + 对应的原型链如下所示：
  ![fig4](https://javascript.info/article/prototype-inheritance/proto-animal-rabbit-chain.png)
+ 原型链中有两个限制：
  + `[[Prototype]]` 之间不能构成闭环引用。如果尝试在一个环中分配 `__proto__`，js 将抛出错误
  + `__proto__` 的值只能是 `null` 或者对象，其它的数据类型都会被忽略
+ 每个对象都只有一个 `[[Prototype]]`，所以对象是不可能继承自两个对象

## 原型链只读
+ 原型链上的对象的属性只能读，而不能写/删除
+ 新添加的属性也只作用于当前调用的对象上
  ```javascript
  let animal = {
    eats: true,
    walk() {
      alert('Animal walk')
    }
  }
  let rabbit = {
    __proto__: animal
  }

  rabbit.walk = function() {
    alert("Rabbit! Bounce-bounce!");
  }

  rabbit.walk()   // "Rabbit! Bounce-bounce!"
  ```
  + 如上代码所示，新添加的 `walk()` 方法仅作用在 `rabbit` 对象上，并没有修改 `animal` 中的 `walk()` 方法
  ![fig1](https://javascript.info/article/prototype-inheritance/proto-animal-rabbit-walk-2.png)
+ 修改当前对象的某个属性时，如果该对象没有该属性，则添加该属性，否则修改该属性的值，并不会作用到其原型链上的这个属性
+ 但是对于新添加的属性如果是原型链上的访问器属性，那么就会执行该访问器的 getter/setter（但是始终不会改变原型链上的内容）
  ```javascript
  let user = {
    name: 'John',
    surname: 'Smith',

    set fullName(value) {
      [this.name, this.surname] = value.split(' ')
    },
    get fullName() {
      return `${this.name} ${this.surname}`
    }
  }
  let admin = {
    __proto__: user,
    isAdmin: true
  }

  admin.fullName    // 'John Smith'（获得 user 的访问属性 fullName）

  admin.fullName = 'Alice Cooper'
  // 执行该行代码后
  admin   // {isAdmin: true, name: "Alice", surname: "Cooper"}
  user    // {name: "John", surname: "Smith"}

  admin.fullName    // 'Alice Cooper'
  ```
  + 从上述代码中，可以看到，执行了 `admin.fullName = 'Alice Cooper'` 后，并没有改变 `user` 中的内容，改变的是 `admin` 的内容
  + `admin` 添加了两个属性：`name` 和 `surname`，这与执行了 `user` 中的 `fullName` setter 有关，`this` 指向了 `admin`，所以给 `admin` 添加了这两个属性
  + 执行 `admin.fullName` 时，同样顺着原型链查找到了 `user` 中的 `fullName` getter，由于 `this` 指向了 `admin`，所以返回的值为 'Alice Cooper'

## “this” 值
+ `this` 的指向并不受原型链的影响，它始终指向被调用的对象
+ **不管是在对象或其原型中找到方法，在进行方法调用时，`this` 始终指向点之前的对象，即被调用的对象**
  + 所以在上节代码示例中，在执行 `admin.fullName` 时，`this` 指向的是 `admin`，而不是 `user`
+ 例如在以下代码中，在 `rabbit` 上执行 `animal` 对象中 `sleep()` 方法时，则给 `rabbit` 创建了 `isSleeping` 属性
  ```javascript
  let animal = {
    walk() {
      // ...
    },
    sleep() {
      this.isSleeping = true
    }
  }
  let rabbit = {
    name: 'white rabbit',
    __proto__: animal
  }

  rabbit.sleep()

  alert(rabbit.isSleeping)    // true
  alert(animal.isSleeping)    // undefined
  ```   
  + 执行 `rabbit.sleep()` 后的结果图如下所示：
  ![fig1](https://javascript.info/article/prototype-inheritance/proto-animal-rabbit-walk-3.png)

+ 当 `this` 所指向的对象的某个属性不存在于该对象上时，依然会顺着原型链进行查找
  ```javascript
  let hamster = {
    stomach: [],
    eat(food) {
      this.stomach.push(food)
    }
  }
  let speedy = {
    __proto__: hamster
  }

  speedy.eat('apple')
  speedy.stomach    // ['apple']
  hamster.stomach   // ['apple']
  ```
  + 在上例中，执行 `speedy.eat('apple')` 时，在原型链上找到了 `hamster` 中的 `eat()`，所以执行该方法，`this` 指向了 `speedy` 对象
  + 但是 `speedy` 对象上并没有 `stomach` 属性，所以又顺着原型链找到了 `speedy` 对象上的 `stomach` 属性，所以最后实际上是往 `speedy.stomach` 添加了 `food`，但是 `this` 始终指向的是 `speedy` 对象


## for...in 循环
+ `for...in` 循环会顺着原型链对所有的属性进行遍历；而 `Object.keys()` 仅获取当前对象的所有属性名，而不会顺着原型链遍历
  ```javascript
  let animal = {
    eats: true
  }
  let rabbit = {
    jumps: true,
    __proto__: animal
  }

  Object.keys(rabbit)   // jumps

  for(let prop in rabbit)
    alert(prop)     // jumps, eats
  ```
+ `obj.hasOwnProperty(key)` 可以用来判断是否只是自身的属性，而不是继承的属性，如果是自身的属性则返回 true
+ `hasOwnProperty` 其实也是 obj 继承得到的方法，从最上层 `Object` 中继承，如下图所示；之所以没有被遍历，只是因为其 `enumerable: false`
  ![fig1](https://javascript.info/article/prototype-inheritance/rabbit-animal-object.png)
+ 所以其他的获得键/值的方法，如 `Object.keys()`、`Object.values()` 都会忽略继承的属性

### 属性查找性能
+ 在现代引擎中，性能方面，不管是从对象还是其原型中获取属性是没有区别的。浏览器会记得找到属性的位置，并在下一个请求中重用它
  ```javascript
  let family = {
    numbers: 4
  }
  let father = {
    name: 'John'
  }
  ```
  + `father.numbers` 和 `family.numbers` 对属性 `numbers` 的访问速度是一样的
  + 对于 `father.numbers` 来说，浏览器会记住是在哪里找到的 `numbers`（即 `family`），那么下次就会从这里开始查找
  + 如果发生变化，浏览器也足够聪明地更新内部缓存，因此优化是安全的