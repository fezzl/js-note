### JS深入之bind的实现
bind()
- 返回一个函数
- 可以传入参数

#### 返回函数的模拟实现
```
Function.prototype.mybind = function(context) {
  var self = this;
  return function() {
    return self.apply(context);
  }
}
```
之所以`return self.apply(context)`是考虑到函数是有可能有返回值
例如
```
var foo = {
  value: 1
};
function bar() {
  return this.value
}
var bindFoo = bar.mybind(foo);
console.log(bindFoo()); // 1 
```
#### 传参的模拟实现
因为bind可以在绑定的时候传递参数，在调用返回的函数的时候也可以传递参数，所以第二版的bind是这样的
```
Function.prototypr.mybind = function(context) {
  var self = this;
  // 获取bind函数除了context之后的所有参数
  var args = Array.prototype.slice.call(arguments,1);
  return function() {
    // 获取再次调用的时候传入的所有参数
    var bindArgs = Array.prototype.slice.call(arguments);
    return self.apply(context, args.concat(bindArgs));
  }
}
```
#### 构造函数的效果模拟实现
因为bind是返回一个函数，所以既然是函数的话，那么也可以作为一个构造函数new出一个实例，所以this会指向这个实例，传入的参数也依然生效
```
Function.prototype.mybind = function(context) {
  var self = this;
  var args = Array.prototype.slice.call(arguments,1);
  var fBound = function() {
    var bindArgs = Array.prototype.slice.call(arguments);
    // 当作为构造函数时，this 指向实例，此时结果为 true，将绑定函数的 this 指向该实例，可以让实例获得来自绑定函数的值
    // 当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
    return self.apply(this instanceof fBound ? this : context,args.concat(bindArgs));
  }
  // 当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
  fBound.prototype = this.prototype;
  return fBound;
}
```
#### 构造函数效果的优化
因为上面的写法是fBound.prototype = this.prototype，所以当改变fBound.prototype的时候this.prototype也会改变，
所以可以用一个空函数当做中转，实现继承this.prototype
```
Function.prototype.mybind = function(context) {
  if(typeof this !== 'function') {
    throw new Error('bind object is not a function')
  }
  var self = this;
  var args = Array.prototype.slice.call(arguments,1);
  var fNop = function() {}
  var fBound = function() {
    var bindArgs = Array.prototype.slice.call(arguments);
    return self.apply(this instanceof fBound ? this : context, args.concat(bindArgs));
  }
  fNop.prototype = this.prototype
  fBound.prototype = new fNop();
  return fBound;
}
```