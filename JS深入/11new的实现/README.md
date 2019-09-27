### JS深入之new的实现
new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象类型之一
，先看看new做了什么，举个例子
```
// Otaku 御宅族，简称宅
function Otaku (name, age) {
    this.name = name;
    this.age = age;

    this.habit = 'Games';
}

// 因为缺乏锻炼的缘故，身体强度让人担忧
Otaku.prototype.strength = 60;

Otaku.prototype.sayYourName = function () {
    console.log('I am ' + this.name);
}

var person = new Otaku('Kevin', '18');

console.log(person.name) // Kevin
console.log(person.habit) // Games
console.log(person.strength) // 60

person.sayYourName(); // I am Kevin
```
从这个例子中，可以看到实例person可以
- 访问到构造函数Otaku的属性
- 访问到构造函数原型Otaku.prototype的属性

因为new是关键字，所以无法像bind一样覆盖函数，所以可以写一个新的函数来模拟new的实现，命名为objectFactory，用法如下
```
function Otaku(...) {

}
// 使用new
var person = new Otaku(...);
// 使用objectFactory
var person = objectFactory(Otaku,...)
```
#### 初步实现
因为new的结果就是返回一个新对象，所以第一步先要创建一个新对象obj，然后改变构造函数的this指向这个新对象obj，可以使用apply改变this的指向obj，Otaku.apply(obj,arguments)来给obj添加新的属性

实例的__proto__会指向构造函数的原型对象，所以实例可以访问到原型上的属性
所以，第一版的实现如下
```
function objectFactory() {
  // 创建一个新对象
  var obj = new Object();
  // 拿到传进来的构造函数
  Constructor = Array.prototype.shift.call(arguments);
  // 新对象的原型指向构造函数的原型对象
  obj.__proto__ = Constructor.prototype;
  // 改变构造函数的this指向obj，给实例添加属性
  Constructor.apply(obj,arguments);
  // 返回这个新的对象
  return obj;
}
```
#### 返回值效果的实现
如果构造函数有返回值，举个例子
```
function Otaku(name,age) {
  this.strength = 60;
  this.age = age;

  return {
    name: name,
    habit: 'Games'
  }
}
var person = new Otaku('Kevin',18);
console.log(person.name) // Kevin
console.log(person.habit) // Games
console.log(person.strength) // undefined
console.log(person.age) // undefined
```
如果构造函数返回一个对象，在实例person中只能访问这个返回对象的属性
如果构造函数返回一个基本类型的值呢
```
function Otaku(name,age) {
  this.strength = 60;
  this.age = age;
  
  return 'something'
}
var person = new Otaku('Kevin',18);
console.log(person.name) // undefined
console.log(person.habit) // undefined
console.log(person.strength) // 60
console.log(person.age) // 18
```
尽管这次又返回值，但是还是按照没有返回值来处理
所以我们需要判断返回值是不是一个对象，如果是对象的话就返回这个对象，如果不是的话就返回obj
第二版如下
```
function objectFactory() {
  // 创建一个新对象
  var obj = new Object();
  // 拿到传进来的构造函数
  Constructor = Array.prototype.shift.call(arguments);
  // 新对象的原型指向构造函数的原型对象
  obj.__proto__ = Constructor.prototype;
  // 改变构造函数的this指向obj，给obj添加属性
  var res = Constructor.apply(obj,arguments);
  // 返回这个新的对象
  return typeof res === 'object' ? res : obj;
}
```