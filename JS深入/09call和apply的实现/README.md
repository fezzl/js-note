### JS深入之call和apply的模拟实现

#### call
call()方法在使用一个指定的this值和若干指定的参数值的前提下调用某个函数的方法
```
var foo = {
  value: 1
};
function bar() {
  console.log(this.value)
}
bar.call(foo); // 1
```
call做了两件事
- 改变了this的指向，指向foo
- bar函数执行了

#### 实现call第一步
在上面的例子中调用call方法，就相当于把foo改造成这样
```
var foo = {
  value: 1,
  bar: function() {
    console.log(this.value)
  }
}
foo.bar(); // 1
```
所以实现call的实现步骤如下
- 将函数设为对象的属性
- 执行该函数
- 删除该函数
以上面的例子就是
```
1、foo.fn = bar
2、foo.fn()
3、delete foo.fn
```
fn是对象的属性名，因为最后也要删掉，所以起什么名都无所谓
第一版的call
```
Function.prototype.mycall = function(context) {
  // 把调用call的函数赋值给指定对象
  context.fn = this;
  context.fn();
  delete context.fn;
}
var foo = {
  value: 1
}
function bar() {
  console.log(this.value);
}
bar.mycall(foo); // 1
```
#### 实现call第二步
因为call函数还能传递参数，但是传递的参数长度不一定，所以可以从Arguments对象读取参数，取出第二个到最后一个参数到数组中
第二版
```
Function.prototype.mycall = function(context) {
  context.fn = this;
  var args = [];
  for(let i = 1; i < arguments.length; i++) {
    args.push('arguments[' + i + ']');
  }
  eval('context.fn(' + args + ')'); // 这里 args 会自动调用 Array.toString() 这个方法
  delete context.fn;
}
var foo = {
  value: 1
};
function bar(name,age) {
  console.log(name);
  console.log(age);
  console.log(this.value);
}
bar.mycall(foo,'zzl',22);
// zzl
// 22
// 1
```
#### 模拟call第三步
现在第二版的call函数还有两个问题
- this参数可以传null，当传的null的时候，this指向window
- 函数是可以有返回值的
第三版
```
Function.prototype.mycall = function(context) {
  context = context || window;
  context.fn = this;
  var args = [];
  for(let i = 1; i < arguments.length; i++) {
    args.push('arguments[' + i + ']');
  }
  var result = eval('context.fn(' + args + ')');
  delete context.fn;
  return result;
}
var value = 1;
var foo = {
  value: 2
};
function bar(name,age) {
  console.log(this.value);
  return {
    name,
    age,
    value:this.value
  }
}
bar.mycall(null); // 1
bar.mycall(foo,'zzl',22);
// 2
// {
  name: 'zzl',
  age: 22,
  value: 2
}
```
#### apply的实现
```
Function.prototype.myapply = function(context,arr) {
  context = context || window;
  context.fn = this;
  var result;
  if(!arr) {
    result = context.fn();
  } else {
    var args = [];
    for(let i = 0;i < arr.length; i++) {
      args.push('arr[' + i + ']');
    }
    result = eval('context.fn(' + args + ')');
  }
  delete context.fn
  return result;
}
```