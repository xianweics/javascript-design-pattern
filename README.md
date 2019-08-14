#  javascript设计模式

## 1 原型继承（面向对象的JavaScript）

### 设计的原理

- 作者为JavaScript设计面向对象系统时，借鉴Self和Smalltalk这两门基于原型的语言。
- 它是基于原型链的委托机制。

### 原型继承基本规则

- 所有的数据都是对象。
	- 除了`undefined`外，一切都应是对象。
- 要得到一个对象，不是通过实例化类，而是找到一个对象作为原型并克隆它。
	- 函数可作为普通函数或者构造器被调用。
	- 当使用`new`来调用函数时，此时函数就是一个构造器，实际上只是先克隆该函数的原型对象。
- 对象会记住它的原型。
		- JavaScript给对象提供一个名为`__proto__`的隐藏属性，某个对象的`__proto__`属性默认会指向它的构造器的原型对象`{constructor}.prototype`。
		- 如果对象本身无法响应某个请求，它会把这个请求委托给它自己的原型。

### 特点：

- 没有提供传统面向对象语言中的类式继承，而是通过原型委托的方式来实现对象与对象之间的继承。
- 没有在语言层面提供对抽象类和接口的支持。
- 是一门典型的动态型语言。动态语言中实现一个原则：面向接口编程，而不是面向实现编程。例如：一个对象若有`push`和`pop`方法，并且这些方法提供了正确的实现，它就可以当作栈来使用。
	
### 鸭子类型

- 含义：如果它走起来像鸭子，叫起来也是鸭子，那么它就是鸭子。
- 作用：指导我们只关注对象的行为，而不关注对象本身，也就是关注has-a，而不是is-a。

### 多态

- 含义：同一操作作用于不同的对象上面，可以产生不同的解释和不同的执行结果。
- 思想：将“做什么“和”谁去做以及怎么去做“分离开来，也就是将”不变的事情“与”可能改变的事物“分离开来。符合开放-封闭原则。
- 作用：通过把过程化的条件分支语句转化为对象的多态性，从而消除这些分支语句。
- JavaScript对象的多态是与生俱来的。
- 例子：
	```javascript
  function makeSound(animal){
    if(animal instanceof Duck){
      console.info('gagaga');
    }else if(animal instanceof Chilcken){
      console.info('gegege');
    }
  /*
  else if(animal instanceof Dog){
    console.info('wangwangwang');
  }
  */
  };

  function Duck(){}
  function Chicken(){}
  // function Dog(){}
  makeSound(new Duck());
  makeSound(new Chicken());
  // makeSound(new Dog());
	```
	优化
  ```javascript
  function makeSound(animal){
    animal.sound();
  };

  function Duck(){}
  Duck.prototype.sound=()=>{
    console.info('gagaga');
  }
  function Chicken(){}  
  Chicken.prototype.sound=()=>{
    console.info('gegege');
  }

  /*
  function Dog(){}  
  Dog.prototype.sound=()=>{
    console.info('wangwangwang');
  }
  */

  makeSound(new Duck());
  makeSound(new Chicken());
  // makeSound(new Dog());
  ```

### 封装

- 含义：一般而言封装指封装数据与封装实现。广义上还包括封装类型和封装变化。
- 目的：将信息隐藏。
- 特点：
	- 使对象内部的变化对其他对象而已是不可见的。对象对它内部的行为负责。
	- 使对象之间的耦合变得松散，对象之间只通过暴露的API接口来通信。
- 封装数据：
	- 例子：
    ```javascript
    const obj = (() => {
      const _name = 'John';
      return {
        getName(){
          return _name;
        }
      }
    })();

    console.info(obj.getName()); // 'John';
    console.info(obj._name); // undefined
    ```
- 封装实现
	- 例子：
    ```javascript
    const each = (arr, cb) => {
      for(let i = 0, l = arr.length; i < l; i++){
        if(cb(i, arr[i]) === false){
          break;
        }
      }
    };

    each([1, 2, 3], (i, item) => {
      if(n > 2){
        return false;
      }
      console.info(i, item);
    });
    ```
	