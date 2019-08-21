#  JavaScript设计模式

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

### 特点

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
  Duck.prototype.sound = () => {
    console.info('gagaga');
  }
  function Chicken(){}  
  Chicken.prototype.sound = () => {
    console.info('gegege');
  }

  /*
  function Dog(){}  
  Dog.prototype.sound = () => {
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

	- 例子
    ```javascript
    const obj = (() => {
      const __name = 'John';
      return {
        getName(){
          return __name;
        }
      }
    })();

    console.info(obj.getName()); // 'John';
    console.info(obj.__name); // undefined
    ```

- 封装实现
	
	- 例子
    ```javascript
    const each = (arr, cb) => {
      for(let i = 0, l = arr.length; i < l; i++){
        if(cb(i, arr[i]) === false){
          break;
        }
      }
    };

    each([1, 2, 3], (i, item) => {
      if(i > 2){
        return false;
      }
      console.info(i, item);
    });
    ```

## 2 this、call、bind和apply

### this

- 特点：基于函数的执行环境绑定的。

- 作为对象的方法调用：`this`指向该对象
  
	- 例子
    ```javascript
    const obj = {
      a: 1,
      getA(){
        console.info(this === obj); // true
        console.info(this.a); // a
      }
    };
    ```
- 作为普通函数调用：`this`指向全局
  
	- 例子
    ```javascript
    window.name = 'global name';
    const obj = {
      name: 'obj name',
      getName(){
        return this.name;
      }
    };

    const getName = obj.getName;
    console.info(getName()); // global name
    ```

- 构造函数调用：`this`通常情况下，指向返回的对象。如果构造器返回一个`object`类型的对象，那么会返回该`object`对象。
  
	- 例子
    ```javascript
    function Fn(){
      this.name = 'my name';
      return {
        name: 'new name'
      }
    }

    const fn = new Fn();
    console.info(fn.name); // new name
    ```

- Function.prototype.call或Function.prototype.apply：可动态改变`this`的指向
  
	- 例子
    ```javascript
    const obj1 = {
      name: 'obj1',
      getName(){
        return this.name;
      }
    };
    const obj2 = {
      name: 'obj2'
    };

    console.info(obj1.getName()); // obj1
    console.info(obj1.getName.call(obj2)); // obj2
    ```

- 修复丢失的this
  
	- 例子
    ```javascript
    const getId = document.getElementById;

    getId('div'); // error: getId is not function
    ```
    
		修正
    
		```javascript
    document.getElementById = ((fn) =>{
      return () => fn.apply(document, arguments);
    })(document.getElementById);

    const getId = document.getElementById;
    console.info(getId('div')); // show div dom
    ```

### bind

- 实现bind
	```javascript
	Function.prototype.bind = function(){
	  var that = this;
	  var context = [].shift.call(arguments);
	  var args = [].slice.call(arguments);
	  return function(){
	    var newArgs = [].slice.call(arguments);
	    return that.apply(context, [].concat(args, newArgs));
	  }
	}
	```

### 数组push分析

- v8源码
	```javascript
	function ArrayPush(){
	  var n = TO_UNIT32(this.length);
	  var m = %_ArgumentsLength();
	  for(var i = 0; i < m; i++){
	    this[i + n] = %_Arguments[i];
	  }
	  this.length = n + m;
	  return this.length;
	}
	```

- 使用Array.prototype.push条件
	
	- 对象本身要可以存取属性
	
	- 对象的`length`属性可读写

- 例子
	```javascript
	const obj = {};
	Array.prototype.push.call(obj, 'first');
	console.info(obj); // { 0: 'first', length: 1 }
	
	const a = 1;
	Array.prototype.push.call(a, 'first');
	console.info(a); // undefined    => 对象本身不能存取属性
	
	function fn(){};
	Array.prototype.push.call(fn, 'first');
	console.info(a); // undefined    => 函数的length是一个只读属性，表示形参的个数
	```

## 3 闭包和高阶函数

### 闭包

- 作用域
  
	- 函数可以用来创造函数作用域。ES6中，使用`let`或`const`可以创造块级作用域。
  
	- 一般情况下，对于全局变量而言，生命周期是永久的，除非主动销毁这个全局变量；而对于函数内用`var`/`const`/`let`关键字声明的局部变量而言，当退出函数时，它们都会随着函数调用的结束而销毁。

- 闭包的作用

	- 保持变量持久化
		
		- 原方式：打印每个对应`div`的index
    	```javascript
      var nodes = document.getElementsByTagName('div');
      for(var i =0; i < nodes.length; i++){
        nodes[i].onclick = function(){
          console.info(i);
        }
      }
      ```
      
			>	无论点击哪个`div`，最后结果都是5。
			
		- 改进：通过闭包把每次循环的`i`值都封闭起来
      ```javascript
      var nodes = document.getElementsByTagName('div');
      for(var i =0; i < nodes.length; i++){
        (function(j){
          nodes[j].onclick = function(){
            console.info(j);
          }})(i);
      }
      ```
      
			> 依次点击`div`，会显示1，2，3，4。
			
	- 封装变量
	
		- 原方式：缓存乘法结果
      ```javascript
      const cache = {};
      function multi(...rest){
        const args = rest.join(',');
        if(args in cache){
          return cache[args];
        }
        let result = 1;
        for(let i = 0; i < rest.length; i++){
          result *= rest[i];
        }
        return cache[args] = result;
      };
      ```
			
			> `cache`变量与`multi`函数一起平行暴露在全局作用域下。
			
		- 改进：把`cache`变量封装到`multi`函数中
      ```javascript
      const multi = (() => {
        const cache = {};
        const calc = (args) => {
            return args.reduce((a, b) => a * b);
        }
        return (...rest) => {
          const args = rest.join(',');
          if(args in cache){
            return cache[args];
          }
          return cache[args] = calc(rest);
        }
      })();
      ```

- 闭包和面向对象设计
	
	- 对象以方法的形式包含了过程，而闭包是在过程中以环境的形式包含了数据；通常用面向对象思想能实现的功能，闭包也能实现，反之亦然。
	
	- 例子
	
		- 闭包
      ```javascript
      const extent = () => {
        let value = 0;
        return {
          call(){
            value++;
            console.info(value);
          }
        }
      }
      const extent1 = extent();
      extent1.call(); // 1
      extent1.call(); // 2
      ```
			
  	- 面向对象
      ```javascript
      const extent = {
        value: 0,
        call(){
           this.value++;
           console.info(this.value);
        }
      };
      extent.call(); // 1
      extent.call(); // 2

      // 或者
      function Extent(){
        this.value = 0;
      }
      Extent.prototype.call = function(){
        this.value++;
        console.info(this.value);
      }
      ```

- 闭包与内存管理
	
	- 把变量放在闭包中和放在全局作用域，对于内存的影响是一致的，不能就说成是内存泄漏，如果将来需要回收这些变量，只需手动把这些变量设置为`null`。
	
	- 跟闭包和内存泄漏有关系的地方：使用闭包的同时容易形成循环引用，如果闭包的作用域链中保存一些DOM的节点，这个时候可能造成内存泄漏。但这本身不是闭包造成的，也并非JavaScript的问题。

### 高阶函数

- 满足的条件
	
	- 函数可以作为参数被传递。例如：回调函数
	
	- 函数可以作为返回值输出

- currying
	
	- 含义：部分求值。一个柯里化函数首先会接受一些参数，接受了这些参数后，该函数并不会立即求值，而是继续返回另一个函数，刚才传入的参数在函数内部形成闭包被保存起来，等函数真正需要求值的时候，之前传入的所有参数会被一次性求值。
	
	- 例子：计算每天的开销
		
		- 无柯里化
			```javascript
			let totalCost = 0;
			function cost(money){
			  totalCost += money;
			}
			cost(100);
			cost(200);
			cost(300);
			console.info(totalCost); // 600
			```
		
		- 柯里化
			```javascript
			function currying(fn){
			  const args = [];
			  return (..rest) => {
			    if(rest.length === 0){
			      return fn(rest);
			    }else{
			      args.push[...rest];
			      return arguments.callee;
			    }
			  }
			}
			function cost(arr){
			  return arr.reduce((a, b) => a + b);
			}
			
			const totalCost = currying(cost);
			totalCost(100);
			totalCost(200);
			totalCost(300);
			console.info(totalCost()); // 600
			```

- uncurrying
	
	- 含义：使本来作为特定对象所拥有的功能的函数可以被任意对象所用，即鸭子类型思想。
	
	- 例子
		```javascript
		Function.prototype.uncurrying = function(){
		  const that = this;
		  return (...rest) => {
		    const context = rest.shift(0,1);
		    return that.apply(context, rest);
		  }
		}
		
		const push = [].push.uncurrying();
		const arr = [1,2,3];
		push(arr, 4);
		console.info(arr);
		```

- 函数节流
	
	- 原理：延迟当前函数的执行，如果该次延迟还没有完成，那么忽略接下来该函数的请求。
	
	- 场景
		
		- `window.onresize`
		
		- `mousemove`
		
		- 上传进度
	
	- 例子
		```javascript
		function throttle(fn, interval = 500){
		  const that = fn;
		  let timer = null;
		  let firstTime = true;
		  
		  return function(){
		    const args = arguments;
		    if(firstTime){
		      that.apply(this, args);
		      return firstTime = false;
		    }
		   if(timer){
		     return false;
		   }
		   timer = setTimeout(()=>{
		     clearTimeout(timer);
		     timer = null;
		     that.apply(this, args);
		   },interval);  
		  }
		}
		
		window.onresize = (() => {
		  console.info('resize');
		}, 300);
		```

- 分时函数
	
	- 原理：分批完成任务。
	
	- 场景：有大量的任务处理，但会影响性能。
	
	- 例子
		
		- 原方式
			```javascript
			let arr = [];
      for (let i = 1; i <= 1000; i++) {
        arr.push(i); //假设arr装载了100个好友数据
      }
      function renderFriendList(data) {
        for (let i = 0, l = data.length; i < l; i++) {
          const div = document.createElement('div');
          div.innerHTML = i;
          document.body.appendChild(div)
        }
      }
      
			renderFriendList(arr)
			```
			
    - 改进：分批处理
      ```javascript
      let arr = [];
      for (let i = 1; i <= 1000; i++) {
        arr.push(i);
      }
      
      function timeChunk(arr, fn, count = 1) {
        let obj;
        let timer = null;
        function start(){
          for (let i = 0; i < Math.min(count, arr.length); i++) {
            fn(arr.shift());
          }
        };
        return () => {
          timer = setInterval(() => {
            if (arr.length === 0) {
              return clearInterval(timer);
            }
            start();
          }, 200)
        }
      }
      timeChunk(arr, (num) => {
        const div = document.createElement('div');
        div.innerHTML = num;
        document.body.appendChild(div);
      }, 8)();
      ```

- 惰性加载函数
	
	- 例子
		
		- 原方式
    	```javascript
      function addEvent(ele, type, handler){
        if(window.addEventListener){
          return ele.addEventListener(type, handler, false);
        }else if(window.attachEvent){
          return ele.attachEvent('on' + type, handler);
        }
      }
      ```
      
			> 存在的问题：当它每次被调用的时候都会执行里面的`if`分支。
    
		- 改进一
			```javascript
      const addEvent = (() => {
        if(window.addEventListener){
          return (ele, type, handler) => ele.addEventListener(type, handler, false);
        }else if(window.attachEvent){
          return (ele, type, handler) => ele.attachEvent('on' + type, handler);
        }
      })();
      ```
      
			> 存在的问题：也许从头到尾都没使用这个函数，但代码加载的时候就立刻执行一次判断。
    
		- 改进二
    	```javascript
    	let addEvent = (ele, type, handler) => {
    	   if(window.addEventListener){
    	    return addEvent = (ele, type, handler) => ele.addEventListener(type, handler, false);
    	  }else if(window.attachEvent){
    	    return addEvent = (ele, type, handler) => ele.attachEvent('on' + type, handler);
    	  }
    	  addEvent(ele, type, handler);
    	};
    	```

## 4 单例模式

- 定义：保证一个类仅有一个实例，并提供一个访问它的全局访问点。

### 面向对象实现单例

- 例子
	```javascript
	function Singleton(name){
	  this.name = name;
	  this.instance = null;
	}
	
	Singleton.getInstance = (name) => {
	   return this.instance || (this.instance = new Singleton(name));
	}
	
	const s1 = Singleton.getInstance('instance1');
	const s2 = Singleton.getInstance('instance2');
	console.info(s1 === s2); // true
	```

### JavaScript中的单例模式

> 传统的单例模式实现在JavaScript中并不适用。

> 单例模式的核心：确保只有一个实例，并且提供全局访问。

> 全局变量不是单例模式，但经常会把全局变量当成单例来使用

> 全局变量存在很多问题，它容易造成命名空间污染。

- 降低全局变量造成命名污染
	
	- 使用命名空间
		
		- 例子
			```javascript
			// 静态命名空间
			const namespace = {
			  a(){},
			  b(){}
			};
			
			// 动态命名空间
			const app = {};
			app.namespace = (name) => {
				const part = name.split('.');
			  let cur = app;
				let hasInclude = true;
			  part.forEach((item) => {
			    if(!cur[item]){
			      hasInclude = false;
			      cur[item] = {};
			    }
					cur = cur[item];
			  });
			  if(hasInclude){
			    throw new Error('Namespace is included');
			  }
			};
			```
  
	- 使用闭包封装私有变量
    
		- 例子
      ```javascript
      const user = (() => {
        const __name = 'name';
        const __age = 10;
        return {
          getUserInfo(){
            return {
              name: __name, 
              age: __age
            };
          }
        };
      })();
      ```

- 通用单例模式
	
	- 例子
  	```javascript
  	function singleton(fn){
  	  let result;
  	  return function(){
  	    return result || (result = fn.apply(this, arguments));
  	  }
  	}
  	```

> 创建对象和管理单例的职责分别放在不同的方法中，这两个方法组合起来才能发挥单例的真正作用。

## 5 策略模式

### 定义

- 定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。

### 目的

- 将算法的使用（不变）与算法的实现分离（可变）开来。

### 组成

- 环境类：接受用户的请求，随后将请求委托给某一个策略类。

- 策略类：封装了具体的算法，并负责具体的计算过程。

### 例子

- 原方式
  ```javascript
  const calcBonus = (level, salary) => {
  	if(level === 'A'){
  		return salary * 4;
  	}
  	if(level === 'B'){
  		return salary * 3;
  	}
  	if(level === 'C'){
  		return salary * 1;
  	}
  };
  
	calcBonus('A', 1000); // 4000
	```
	
	- 存在的问题
		
		- `calcBonus`函数比较大，包含较多的`if-else`。
		
		- 函数违法开放-封闭性原则。
		
		- 算法复用性差

- 改善
	
	- 例子：面向对象的实现
  	```javascript
    // 策略类
    function performanceA(){};
    performanceS.prototype.calc = (salary) => {
      return salary * 4;
    };

    function performanceB(){};
    performanceB.prototype.calc = (salary) => {
      return salary * 3;
    };

    function performanceC(){};
    performanceC.prototype.calc = (salary) => {
      return salary * 1;
    };

    // 环境类
    function Bonus(){
      this.salary = null;
      this.strategy = null;
    };
    Bonus.prototype.setSalary = function(salary){
      this.salary = salary;
    };
    Bonus.prototype.setStrategy = function(strategy){
      this.strategy = strategy;
    };
    Bonus.prototype.getBonus = function(){
      return this.strategy.calc(this.salary);
    };

    const bonus = new Bonus();
    bonus.setSalary(1000);
    bonus.setStrategy(new performanceA());
    console.info(bonus.getBonus()); // 4000
    ```
	
	- 例子：JavaScript模式
		```javascript
		const strategies = {
		  'A': (salary) => salary * 4,
		  'B': (salary) => salary * 3,
		  'C': (salary) => salary * 1,
		}；
		const calcBonus = (level, salary) => {
		  return strategies[level](salary);
		};
		console.info(calcBonus('A', 1000)); // 4000
		```

### 优点

- 利用组合、委托和多态等思想，可以有效地避免多重条件语句。

- 符合开放-封闭原则，易于切换、理解、扩展。

- 复用性强，避免重复的复制粘贴。

### 缺点

- 增加许多策略类或者策略对象。

- 必须了解所有的策略类，并且了解各个策略之间的不同点，这样才能选择合适的策略。


















