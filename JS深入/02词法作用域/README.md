### JS深入之词法作用域

#### 作用域
- 作用域指定义变量的一个区域
- 作用域规定了如何查找变量，也就是确定了当前执行代码对变量访问权限
- JavaScript采用的是词法作用域，也就是静态作用域

看一下下面这个例子
```
var value = 1;

function foo() {
  console.log(value);
}

function bar() {
  var value = 2;
  foo();
}
bar()
```
上面代码的执行结果是打印1，因为JavaScript采用的是词法作用域，也就是在执行foo函数时，会在他定义的函数内部寻找变量value,如果没找到，会去这个函数的上一级去寻找，也就是全局作用域去寻找变量value,最后打印的就是1

```
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
```

```
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();
```
上面两段代码的执行结果都是`local scope`,因为JavaScript采用的是词法作用域，函数的作用域基于函数创建的位置

JavaScript 函数的执行用到了作用域链，这个作用域链是在函数定义的时候创建的。嵌套的函数 `f()`定义在这个作用域链里，其中的变量 scope 一定是局部变量，不管何时何地执行函数 `f()`，这种绑定在执行 `f()` 时依然有效。