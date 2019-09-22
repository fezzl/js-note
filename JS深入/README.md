### JS深入之原型到原型链

####  构造函数创建对象
```
function Person () {

}
var person = new Person();
person.name = 'zzl';
```
在这个例子中，通过`new`一个构造函数创造出一个实例对象`person`

#### prototype
每个函数都有一个`prototype`属性
```
function Person() {

}
// 注意: prototype是函数才有的属性
Person.prototype.age = 22;
var person1 = new Person();
var person2 = new Person();
person1.name = 'zzl';
person2.name = 'zhangsan';
```
这个函数的`prototype`究竟指向什么呢？其实它指向的构造函数实例的原型对象，也就是上面例子中`person1`,`person2`的原型对象

什么才是原型对象呢？你可以这样理解: 每一个`JavaScript`对象(除了`null`)在创建的时候都会关联一个对象，这个对象就是它的原型对象，这个对象会继承它的原型对象上的属性，比如上面例子中的`person1.age`，`person2.age`都是继承原型对象上的属性

#### `__proto__`
了解了函数与原型对象之间的关系后，你会想实例`person`和原型对象`Person.prototype`之间会有关系吗，实例和原型对象之间是通过`__proto__`关联的
```
function Person() {

}
var person = new Person();
console.log(person.__proto__ === Person.prototype) // true
```
现在,构造函数`Person`和实例`person`,都可以指向原型对象`Person.prototype`,那么原型对象可以指向构造函数和实例吗

#### constructor
指向实例倒是没有，因为实例可以通过构造函数创造多个实例，但是原型对象可以通过`constructor`属性指向构造函数
```
function Person() {

}
var person = new Person();
console.log(Person.prototype.constructor === Person); // true
```

#### 实例与原型
当读取实例中的某个属性时，会先从实例本身查找，如果实例本身没有这个属性的话，会从它的原型对象上查找，如果还是查不到的话就会去原型的原型上查找，一直到最顶层为止
```
function Person() {

}
Person.prototype.name = 'zhangsan';
var person = new Person();

person.name = 'zzl';
console.log(person.name); // zzl
delete person.name;
console.log(person.name); // zhangsan
```
在上面的例子中，我们给实例对象一个`name`属性，当我们打印实例对象的`name`属性时，自然就是`zzl`

但是当我们删除实例上的name属性时，再去打印`person.name`时，实例对象中找不到name属性的话就会去查找实例的原型`person.__proto__`，也就是`Person.prototype`,最后我们在这个原型对象上找到了这个name属性的值`zhangsan`

但是在原型上也找不到呢，原型是否也有原型

#### 原型的原型
上面说到只要是对象(除了null),就会有它的原型对象，因为原型对象也是对象，所以原型对象也有自己的原型对象，`Person.prototype`原型对象指向他构造函数的prototype，`Person.prototype`的构造函数是Object，所以:
```
function Person() {

}
console.log(Person.prototype.__proto__ === Object.prototype) // true
```

#### 原型链
Object.prototype的原型对象是什么呢，我们打印一下试试:
```
console.log(Object.prototype.__proto__) // null
```
这表明了`Object.prototype`没有原型
所以在查找属性时查找到`Object.prototype`就可以停止查找了

原型链: 就是通过__proto__连接实例对象的所有原型对象，原型组成的链状结构就是原型链
