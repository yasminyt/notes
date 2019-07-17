# 私有 & 受保护的属性和方法
+ 私有属性和方法，是指只能在类的内部访问和使用的属性和方法，外部不能访问，这有利于代码的封装（但是ES6并没有提供）

## 受保护的属性
+ 通常用下划线 `_` 开头的命名来表示受保护的属性
+ 对于 protected 属性来说，一般通过 getter/setter 来对其进行修改和访问
+ 如果想要 protected 属性只读，那么只需要定义它的 getter 即可
  ```javascript
  class CoffeeMachine {
    constructor(power) {
      this._power = power;
    }
    get power() {
      return this._power;
    }
  }
  // create the coffee machine
  let coffeeMachine = new CoffeeMachine(100);
  alert(`Power is: ${coffeeMachine.power}W`); // Power is: 100W
  coffeeMachine.power = 25; // Error (no setter)
  ```
+ 但是如果想要对 protected 属性做更多的操作时，getter/setter 是不够的（getter 只能返回，setter 只能接受一个参数），所以一般来说是自定义 getter/setter 方法，如下例所示：
  ```javascript
  class CoffeeMachine {
    _waterAmount = 0;

    // 自定义一个 set 方法
    setWaterAmount(value) {
      if (value < 0) throw new Error("Negative water");
      this._waterAmount = value;
    }

    // 自定义一个 get 方法
    getWaterAmount() {
      return this._waterAmount;
    }
  }

  new CoffeeMachine().setWaterAmount(100);
  ```
  + 这个根据实际的需要来选择
+ 实际上 protected 属性也是可以被正常访问和修改的，只是语义上的受保护，并没有得到语言支持

## 私有属性和方法
+ 使用 `#` 开头的命名来表示私有属性和方法
+ 但是这只是个提案，并没有成为标准中的内容，所以 js 引擎是不支持的

### See more from [阮一峰的ES6入门-私有属性](http://es6.ruanyifeng.com/#docs/class#%E7%A7%81%E6%9C%89%E6%96%B9%E6%B3%95%E5%92%8C%E7%A7%81%E6%9C%89%E5%B1%9E%E6%80%A7)