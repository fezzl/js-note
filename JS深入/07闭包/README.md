### JS深入闭包
闭包: 是指那些能够访问自由变量的函数
自由变量: 自由变量是指在函数中使用，但既不是函数参数也不是函数的局部变量的变量
闭包 = 函数 + 函数能够访问的自由变量

ECMAScript中，闭包是:
1.从理论角度：所有的函数。因为它们都在创建的时候就将上层上下文的数据保存起来了。哪怕是简单的全局变量也是如此，因为函数中访问全局变量就相当于是在访问自由变量，这个时候使用最外层的作用域
2.从实践角度：以下函数才算是闭包
- 即使创建它的上下文已经被销毁，它仍然存在(内部函数从父函数中返回)
- 在代码中引用了自由变量

```
var data = [];

for (var i = 0; i < 3; i++) {
  data[i] = function () {
    console.log(i);
  };
}

data[0]();
data[1]();
data[2]();
```
这个答案都是3
当执行到`data[0]`函数的之前，此时全局上下文的VO为：
```
globalContext = {
  VO: {
    data: [...],
    i: 3
  }
}
```
当执行`data[0]`函数的时候，此时的`data[0]`函数的作用域链为
```
data[0]Context = {
  Scope: [AO, globalContext.VO]
}
```
`data[0]Context`的AO没有i，就会到`globalContext.AO`中查找，i为3，所以打印的结果是3，`data[1]`和`data[2]`也是一样

所以可以改成闭包
```
var data = [];

for (var i = 0; i < 3; i++) {
  data[i] = (function (i) {
        return function(){
            console.log(i);
        }
  })(i);
}

data[0]();
data[1]();
data[2]();
```
当执行到`data[0]`函数之前，全局上下文的VO为
```
globalContext = {
  AO: {
    data: [...],
    i: 3
  }
}
```
当执行到`data[0]`函数的时候，`data[0]`的作用域链是
```
data[0]Context = {
  Scope: [AO, 匿名函数Context.AO, globalContext.VO]
}
```
匿名函数执行上下文AO为：
```
匿名函数Context = {
  AO: {
    arguments: {
      0: 0,
      length: 1
    }
    i: 0
  }
}
```
`data[0]Context`的AO没有i值，所以会沿着作用域链从匿名函数Context.AO查找，这时候就可以找到i的值为0，找到了之后就不会往globalContext上找了，即使globalContext上也有i值3，所以打印的结果为0
`data[1]`和`data[2]`也是一样的道理