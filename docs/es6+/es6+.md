#### Let & Const

* 作用域 基础语法

  + 全局作用域 window.$
  + 函数作用域 function fn(){}
  + 块级作用域 let 块级作用域不会挂载到window对象，不能作属性直接使用
  + 动态作用域 bind

#### for虚幻支持break和continue arr.every需要返回true才会继续执行

#### 快速生成数组

*  **Array.from 类数组转为数组**

  + **Array.from(**`arrayLike|iterable, ?mapFn, ?thisArg`**)**; 
  + `({0: 'a', 1: 'b', 2: '3', length: 4} 这个就是类数组)` 
  + `Array.from({length: 5}, () => 1); 返回 => [1, 1, 1, 1, 1]` 

* Array.of() 快速生成一个数组

  + `Array.of(1, 2, 3, 4, 5, 6, 7); 返回 =>  [1, 2, 3, 4, 5, 6, 7]` 

* (**修改原数组**) Array.fill(value, start|0, end|array.length) 填充数组的默认值其实也可以修改

  + `Array(5).fill(1); ; 返回 => [1, 1, 1, 1, 1]` 
  + `Array(3).fill(); ; 返回 => [undefined, undefined, undefined]` 
  + `let arr = [1, 2, 3, 4, 5]; arr.fill(6, 2) ; 返回 => [1, 2, 6, 6, 6]` 

#### 快速查找数组中是否包含某个值

* **Array.filter(item => item % 2 === 0)** 找所有满足的值 直到结束 如果数据量特别的大性能就比较差因为会遍历所有的数组对象

  + `let arr = [1, 2, 3, 4, 5]; arr.filter(item => item % 2 === 0); 返回 => [2, 4]` 
  + `let arr = [1, 2, 3, 4, 5]; arr.filter(item => item % 2 === 3); 返回 => []` 

* **Array.find(item => item === 2)** 找到满足的第一个值 即 停止

  + `let arr = [1, 2, 3, 4, 5]; arr.find(item => item === 3); 返回 => 3(数字3，即数组内容而非索引)` 
  + `let arr = [1, 2, 3, 4, 5]; arr.find(item => item === 6); 返回 => undefined` 

* **Array.findIndex(item => item === 2)** 找到满足的第一个值的索引下标 即 停止

  + `let arr = [1, 2, 3, 4, 5]; arr.findIndex(item => item%3 === 0); 返回 => 2` 

#### 定义一个类Class

* es2015写法    

``` javascript
  // 这样写法有问题 即 每个实例对象都会挂载一个eat方法导致实例化对象过于庞大
  var Animal = function(type) {
      this.type = type;
      this.eat = function() {
          console.log('eat fn')
      }
  }
  console.log(new Animal('dog'));
  //返回 => {
  //     type: 'dog',
  //     eat: fn()
  // };
  console.log(new Animal('cat'));
  //返回 => {
  //    type: 'cat',
  //    eat: fn()
  //};

  // 正确写法
  var Animal = function(type) {
      this.type = type;
  }

  Animal.prototype.eat = function() {
      // 这里可以理解为 挂载方法在树根上 而非挂载在树杈上
      console.log('eat');
  }

  // 修改eat方法
  dog.constructor.prototype.eat = function() {
      console.log('change public eat')
  }
```

* es6+写法

``` javascript
  class Animal {
      constructor(type) {
          this.type = type
      }

      eat() {
          console.log('eat');
      }
  }

  let dog = new Animal('dog');
  console.log(dog.eat);

`console.log(typeof(Animal)) => function 实际上Class就是es2015写法的语法糖 实际就是构建了一个function 并且把方法挂载到了树根 prototype上面` 

  // 保护变量 实际上是闭包原理 外界传一个_age 然后内部拿到赋值age但是实际上外部实例化对象是拿不到_age的值 从而变为一个私有属性
  let _age = 18; // 解析getter setter
  class Animal {
      constructor(type) {
          this.type = type
      }

      // 这里要强调一下 如果一个看似函数的前面有getter setter 那么这个属性会被放到类的最顶层(也就是constructor外面，constructor的上面) 并且会变为一个属性
      get age() { // 如果只有一个get那么就是只读属性
          return _age
      }

      set age(val) {
          _age = val
      }

      eat() {
          console.log('eat');
      }
  }

  let dog = new Animal('dog');
  console.log(dog.age);
  dog.age = 5;
  console.log(dog.age);
```

* 类的静态方法

  + es2015写法

``` javascript
  let Animal = function(type) {
      this.type = type
  }

  Animal.prototype.eat = function() {
      Animal.run(); // 如果改为this.run()这个地方会报错 因为静态方法在实例对象上面是找不到的
      console.log('eat')
  }

  Animal.run = function() {
      console.log('running');
  }

  let dog = new Animal('dog');
  dog.eat();
  dog.run(); // 这个地方会报错 因为静态方法在实例对象上面是找不到的
```

  + es6+写法

``` javascript
  class Animal {

      constructor(type) {
          this.type = type
      }

      eat() {
          Animal.run();
          console.log('eat');
      }

      static run() {
          console.log('running');
      }
  }

  let dog = new Animal('dog');
  dog.eat();
  dog.run(); // 这个地方会报错 因为静态方法在实例对象上面是找不到的
```

* **什么时候使用类的静态方法什么时候用实例对象方法呢？**
  + 当方法**依赖**于类的属性或方法就必须定义为类的实例对象方法
  + 当方法**不依赖**于类的属性或方法就必须定义为类的静态方法
  + **实际上就是类的静态方法获取不到当前的实例对象**
