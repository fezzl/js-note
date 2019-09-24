### JS深入之执行上下文

#### 顺序执行
先来看看下面这两段代码的执行结果
```
var foo = function() {
  console.log('foo1');
}
foo(); // foo1
var foo = function() {
  console.log('foo2');
}
foo(); // foo2
```

```
function foo() {
  console.log('foo1');
}
foo(); // foo2
function foo() {
  console.log('foo2);
}
foo(); // foo2
```
因为JS引擎不是一行一行的执行代码的，而是一段一段地分析执行。当执行一段代码的时候，会进行一个”准备工作“，比如第一个例子中变量提升，和第二个例子的函数提升

#### 可执行代码
JS的可执行代码有三种
- 全局代码
- 函数代码
- eval代码
当我们执行一个函数的时候，就会进行准备工作，这里的准备工作可以叫做”执行上下文“

#### 执行上下文栈
如何管理创建那么多的执行上下文呢？JS引擎创建了执行上下文栈(Execution context stack，ECS)来管理执行上下文，为了模拟执行上下文的行为，让我们定义执行上下文栈是一个数组
```
ECStack = [];
```
当JS要开始执行代码的时候，最先遇到的是全局代码，所以初始化的时候首先会向执行上下文栈压入一个全局执行上下文，我们用globalContext表示它，并且只有当整个应用程序结束的时候，ECStack才会被清空，所以在程序结束之前，ECStack最底部永远有个globalContext
```
ECStack = [
  globalContext
]
```
当JS遇到下面的这段代码
```
function fun3() {
  console.log('fun3);
}
function fun2() {
  fun3();
}
function fun1() {
  fun2();
}
fun1();
```
当执行一个函数的时候，就会创建一个执行上下文，并且压入执行上下文栈，当函数执行完之后就会将执行上下文从栈中弹出，知道这个工作原理后再来理解上面这段代码
```
// 伪代码

// fun1()
ECStack.push(<fun1> functionContext);

// fun1中竟然调用了fun2，还要创建fun2的执行上下文
ECStack.push(<fun2> functionContext);

// 擦，fun2还调用了fun3！
ECStack.push(<fun3> functionContext);

// fun3执行完毕
ECStack.pop();

// fun2执行完毕
ECStack.pop();

// fun1执行完毕
ECStack.pop();

// javascript接着执行下面的代码，但是ECStack底层永远有个globalContext
```