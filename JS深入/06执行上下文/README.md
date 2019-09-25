### JS深入执行上下文
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
1、执行全局代码，创建全局上下文，全局上下文被推入执行栈
```
ECStack = [
  globalContext
];
```
2、全局上下文初始化
```
globalContext = {
  VO: [global],
  Scope: [globalContext.VO],
  this: globalContext.VO
}
```
3、初始化的同时，checkscope函数被创建，保存作用域链到函数内部属性\[[scope]]
```
checkscope.[[scope]] = {
  globalContext.VO
}
```
4、执行checkscope函数，创建checkscope函数的执行上下文，checkscope执行上下文被压入到执行上下文栈
```
ECStack = [
  checkscopeContext,
  globalContext
]
```
5、checkscope函数执行上下文初始化
- 复制函数的\[[scope]]属性创建作用域链
- 用arguments创建活动对象
- 初始化活动对象，即加入形参，函数声明，变量声明
- 将活动对象压入checkscope作用域链顶端
同时f函数被创建，把f函数的作用域链保存到\[[scope]]
```
checkscopeContext = {
  AO: {
    arguments: {
      length: 0
    },
    scope: undefined,
    f: reference to function f(){}
  },
  Scope: [AO, globalContext.VO],
  this: undefined
}
```
6、执行完checkscope后，把checkscope从执行上下文栈中弹出
```
ECStack = [
  globalContext
];
```
7、执行f函数，创建f函数的执行上下文，把f的执行上下文压入到执行上下文栈中
```
ECStack = [
  fContext,
  globalContext
]
```
8、f函数执行上下文初始化
- 复制函数\[[scope]]的属性创建作用域链
- 用arguments创建活动对象
- 初始化活动对象，即加入形参，函数声明，变量声明
- 将活动对象压入f函数的作用域链的顶端
```
fContext = {
  AO: {
    arguments: {
      length: 0
    }
    scope: undefined
  }
  Scope: [AO, checkscopeContext.AO, globalContext.VO],
  this: undefined
}
```
9、f函数执行，沿着作用域查找scope，返回scope值
10、f函数执行完之后，从执行上下文栈中弹出
```
ECStack = [
  globalContext
]
```