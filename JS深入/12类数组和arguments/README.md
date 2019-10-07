### JS深入类数组对象与arguments
类数组对象：
拥有一个 length 属性和若干索引属性的对象

#### Arguments对象
Arguments 对象只定义在函数体中，包括了函数的参数和其他属性。在函数体中，arguments 指代该函数的 Arguments 对象。

#### length属性
```
function foo(b, c, d){
    console.log("实参的长度为：" + arguments.length)
}

console.log("形参的长度为：" + foo.length)

foo(1)

// 形参的长度为：3
// 实参的长度为：1
```
#### callee属性
Arguments 对象的 callee 属性，通过它可以调用函数自身
```
var data = [];

for (var i = 0; i < 3; i++) {
    (data[i] = function () {
       console.log(arguments.callee.i) 
    }).i = i;
}

data[0]();
data[1]();
data[2]();

// 0
// 1
// 2
```
#### arguments和对应参数的绑定
```
function foo(name, age, sex, hobbit) {

    console.log(name, arguments[0]); // name name

    // 改变形参
    name = 'new name';

    console.log(name, arguments[0]); // new name new name

    // 改变arguments
    arguments[1] = 'new age';

    console.log(age, arguments[1]); // new age new age

    // 测试未传入的是否会绑定
    console.log(sex); // undefined

    sex = 'new sex';

    console.log(sex, arguments[2]); // new sex undefined

    arguments[3] = 'new hobbit';

    console.log(hobbit, arguments[3]); // undefined new hobbit

}

foo('name', 'age')

```
传入的参数，实参和 arguments 的值会共享，当没有传入时，实参与 arguments 值不会共享
#### 传递参数
将参数从一个函数传递到另一个函数
```
// 使用 apply 将 foo 的参数传递给 bar
function foo() {
    bar.apply(this, arguments);
}
function bar(a, b, c) {
   console.log(a, b, c);
}

foo(1, 2, 3)
```
#### 强大的ES6
使用ES6的 ... 运算符，我们可以轻松转成数组
```
function func(...arguments) {
    console.log(arguments); // [1, 2, 3]
}

func(1, 2, 3);
```
#### 应用
- 参数不定长
- 函数柯里化
- 递归调用
- 函数重载