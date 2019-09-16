## js高级教程笔记

### 五、引用类型

数组
`instanceof` 的问题
它假定只有一个全局环境,如果在网页中包含多个框架的话，那实际上就存在多个全局环境，如果从一个全局环境中传一个数组到另一个全局环境，传入的数组的构造函数与这个函数原生的构造函数不同。
`Array.isArray`不管在哪个环境都可以判断是否是数组

---

正则
元字符必须转义，元字符包括`( [ { \ ^ $ | ) ? * + .]}`

函数
在函数内部，有两个特殊的对象:arguments 和 this， arguments是一个类数组对象,包含着传入函数中的所有参数.这个对象还有一个callee属性,该属性是一个指针,指向拥有这个 arguments 对象的函数,他可应用在递归上
```
function factorial(num){
    if (num <=1) {
        return 1;
    } else {
        return num * arguments.callee(num-1)
} }
```
this 引用的是函数据以执行的环境对象

caller 这个属性中保存着调用当前函数的函数的引用

### 六、面向对象

#### 数据属性和访问器属性
1、数据属性
- \[[Configurable]]:表示能否通过 delete 删除属性从而重新定义属性，能否修改属性的特
性，或者能否把属性修改为访问器属性。像前面例子中那样直接在对象上定义的属性，它们的
这个特性默认值为 true
- \[[Enumerable]]:表示能否通过 for-in 循环返回属性。像前面例子中那样直接在对象上定
    义的属性，它们的这个特性默认值为 true
- \[[Writable]]:表示能否修改属性的值。像前面例子中那样直接在对象上定义的属性，它们的这个特性默认值为true
- \[[Value]]:包含这个属性的数据值。读取属性值的时候，从这个位置读;写入属性值的时候，
把新值保存在这个位置。这个特性的默认值为 undefined

修改属性默认的特性，必须使用 ECMAScript 5 的 `Object.defineProperty()`方法。
这个方法 接收三个参数:属性所在的对象、属性的名字和一个描述符对象。描述符(descriptor)对象的属性必须是:configurable、enumerable、writable 和 value。设置其中的一或多个值，可以修改对应的特性值
```
var person = {};
Object.defineProperty(person, "name", {
    writable: false,
    value: "Nicholas"
});
alert(person.name); //"Nicholas"
person.name = "Greg"; 
alert(person.name); //"Nicholas"

```
2、访问器属性
它们包含一对儿getter和setter函数
- \[[Configurable]]:表示能否通过 delete 删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为数据属性。对于直接在对象上定义的属性，这个特性的默认值为 true
- \[[Enumerable]]:表示能否通过 for-in 循环返回属性。对于直接在对象上定义的属性，这个特性的默认值为 true
- \[[Get]]:在读取属性时调用的函数。默认值为 undefined
- \[[Set]]:在写入属性时调用的函数。默认值为 undefined
```
var book = {
    _year: 2004,
    edition: 1 
};
Object.defineProperty(book, "year", {
    get: function(){
        return this._year;
    },
    set: function(newValue){
        if (newValue > 2004) {
            this._year = newValue;
            this.edition += newValue - 2004;
        }
    }
});
book.year = 2005; 
alert(book.edition);
```
new 操作符做了什么
1.创建一个新的对象
2.把构造函数的作用域指向这个新的对象,this指向这个新的对象
3.执行构造函数的代码
4.返回新的对象

#### 原型模式
- 每个函数都有一个 prototype(原型)属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法

- 使用 for-in 循环时，返回的是所有能够通过对象访问的、可枚举的(enumerated)属性，其中 既包括存在于实例中的属性，也包括存在于原型中的属性

- 要取得对象上所有可枚举的实例属性，可以使用 ECMAScript 5 的 `Object.keys()`方法。这个方法接收一个对象作为参数，返回一个包含所有可枚举属性的字符串数组

- 如果你想要得到所有实例属性，无论它是否可枚举，都可以使用 `Object.getOwnPropertyNames()` 方法。

#### 继承
##### 原型链继承(缺点: 继承父类的引用类型会造成数据的共享)
通过实现原型链，本质上是扩展原型搜索机制。当以读取模式访问一个实例属性时，首先会在实例中搜索该属性。如果没有找到该属性，则会继续搜索实例的原型。在通过原型链实现继承的情况下，搜索过程就得以沿着原型链继续向上
确定原型和实例的关系
- 使用`instanceof`来测试实例与原型链中出现过的构造函数，结果会返回true
- 使用`isPrototypeOf()`,只要是原型链中出现过的原型,都可以说是该原型链所派生的实例的原型

##### 构造函数继承(缺点: 父类中原型上的属性和方法子类继承不到)
在子类的构造函数中调用父类的构造函数，并且可以把参数传给父类，为了确保父类的构造函数不会重写子类型的属性，可以在调用父类构造函数后，再添加子类型的属性

##### 组合继承(缺点: 父类型调用了两次)
使用原型链实现对原型属性的方法的继承，而通过借用构造函数来实现对实例属性的继承。这样既通过在原型上定义方法实现了函数复用，又能保证每个实例都有它自己的属性
```
function SuperType(name){
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function(){
  alert(this.name);
}
function SubType(name,age) {
  // 继承属性
  SuperType.call(this,name); // 调用第二次
  this.age = age;
}
SubType.prototype = new SuperType(); // 调用第一次
SubType.prototype.constructor = SubType;
Subtype.prototype.sayAge = function () {
  alert(this.age)
}
var instance1 = new SubType('Nicholas',29);
instance1.colors.push('black'); // 'red,blue,green,black'
instance1.sayName(); // 'Nicholas'
instance1.sayAge(); // 29

var instance2 = new SubType('Greg', 27);
alert(instance2.colors); // 'red,blue,green';
instance2.sayName(); // 'Greg'
instance2.sayAge(); // 27
```

#### 原型式继承
Object.create()方法规范了原型式继承,这个方法接收两个参数: 一个用作新对象原型的对象和（可选的）一个为新对象定义额外属性的对象。
```
var person = {
  name: 'Nicholas',
  firends: ["Shelby", "Court", "Van"]
}

var anotherPerson = Object.create(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerson = Object.create(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");
alert(person.friends); //"Shelby,Court,Van,Rob,Barbie"
```

#### 寄生组合继承
不必为了指定子类型的原型而调用超类型的构造函数，我们所需要的无非就是超类型 原型的一个副本而已
```
function SuperType(name){
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function(){
  alert(this.name);
}
function SubType(name,age) {
  // 继承属性
  SuperType.call(this,name);
  this.age = age;
}
SubType.prototype = Object.create(SuperType.prototype);
SubType.prototype.constructor = SubType;
Subtype.prototype.sayAge = function () {
  alert(this.age);
}
var instance1 = new SubType('Nicholas',29);
instance1.colors.push('black'); // 'red,blue,green,black'
instance1.sayName(); // 'Nicholas'
instance1.sayAge(); // 29

var instance2 = new SubType('Greg', 27);
alert(instance2.colors); // 'red,blue,green';
instance2.sayName(); // 'Greg'
instance2.sayAge(); // 27

```

### 七、函数表达式
提升: 使用函数声明的方式定义函数会有函数提升，而使用函数表达式则没有

闭包: 指有权访问另一个函数作用域的变量的函数(作用域链:作用域链本质上是一个指向变量对象的指针列表，它只引用但不实际包含变量对象)
一般来说，函数在执行完之后就会销毁函数，包括销毁函数的局部变量对象，但是因为闭包的存在，闭包的作用域链会引用外部函数的活动对象，所以当函数执行完之后。函数执行环境的作用域链会被销毁，但是函数的变量对象不会被垃圾回收，直到匿名函数被销毁后才会释放外部函数的活动对象(或者通过给匿名函数解除引用,设置null)

闭包的缺点: 由于闭包会携带包含它的函数的作用域，因此会比其他函数占用更多的内存。过度使用闭包会造成内存泄露
```
function createFunctions(){
  var result = new Array();
  for (var i=0; i < 10; i++){
    result[i] = function(){
      return i;    
    };
  }
  return result;
}
```
返回一个函数数组，执行每个函数返回的值都是10，因为它们都引用同一个变量对象i，当外部函数执行完之后i的值就变成10，所以在每个函数内部i的值都是10

```
function createFunction () {
  var result = new Array();
  for(var i = 0; i < 10; i++) {
    result[i] = function(num) {
      return function () {
        return num;
      }
    }(i)
  }
  return result
}
```
我们没有直接把闭包赋值给数组，而是定义了一个匿名函数，并将立即执行该匿名函数的结果赋给数组。这里的匿名函数有一个参数 num，也就是最终的函数要返回的值。在调用每个匿名函数时，我们传入了变量 i。由于函数参数是按值传递的，所以就会将变量 i 的当前值复制给参数 num。而在这个匿名函数内部，又创建并返回了一个访问 num 的闭包。这样一来，result 数组中的每个函数都有自己 num 变量的一个副本，因此就可以返回各自不同的数值了

this: 在运行时基于函数的执行环境绑定的
```
var name = "The Window";
var object = {
    name : "My Object",
    getNameFunc : function(){
        return function(){
            return this.name;
        };
    } 
};
alert(object.getNameFunc()()); //"The Window"(在非严格模式下)
```
每个函数被调用时都会取得两个特殊变量: this和arguments。内部函数在搜索这两个变量时，只会搜索到其活动对象，因此永远无法直接访问外部函数中的这两个变量。不过，可以把外部作用域中的this对象保存在一个闭包可以访问到的变量里，就可以让闭包访问该对象

```
var name = "The Window";
var object = {
    name : "My Object",
    getNameFunc : function(){
        var that = this;
        return function(){
            return that.name;
        };
    } 
};
alert(object.getNameFunc()()); // "My Object"

```
块级作用域: 使用闭包可以在 JavaScript 中模仿块级作用域
```
  (function(){
    //这里是块级作用域 
  })();
```
### 十三、事件

#### 事件流
事件流描述的是从页面接收事件的顺序(事件捕获 -> 目标阶段 -> 冒泡阶段)

#### 事件冒泡
事件开始时由最具体的元素(文档中嵌套层次最深的那个节点)接收，然后逐级向上传播到较为不具体的节点(文档)
element -> body -> html -> document -> window

#### 事件捕获(老版本浏览器不支持)
事件从最上层的节点向下传递，最后接收的节点就是目标节点
window -> document -> html -> body -> element

#### DOM0级事件
每个元素(包括 window 和 document)都有自己的事件处理程序属性，这些属性通常全部小写， 例如 onclick。将这种属性的值设置为一个函数，就可以指定事件处理程序

#### DOM2级事件——可以添加多个事件处理程序
DOM2 级事件定义了两个方法，用于处理指定和删除事件处理程序的操作:addEventListener() 和 removeEventListener()。所有 DOM 节点中都包含这两个方法，并且它们都接受 3 个参数:要处理的事件名、作为事件处理程序的函数和一个布尔值。最后这个布尔值参数如果是 true，表示在捕获阶段调用事件处理程序;如果是 false，表示在冒泡阶段调用事件处理程序

### 二十、JSON
JSON是一种数据格式，不是一种编程语言
JSON对象有两个方法: stringify()和parse(),这两个方法分别用于把JavaScript对象序列化为JSON字符串和把JSON字符串解析为原生JavaScript值

### 二十一、Ajax——无须刷新页面即可从服务器取得数据
核心: XMLHttpRequest对象(XHR)
```
var xhr = new XMLHttpRequest()
```

#### XHR的用法
在使用 XHR 对象时，要调用的第一个方法是 `open()`,它接收3个参数: 要发送的请求的类型、请求的URL和表示是否异步发送请求的布尔值,调用 `open()` 方法并不会真正发送请求，而只是启动一个请求以备发送，要发送特定的请求, 必须调用 `send()` 方法
```
 xhr.open("get", "example.php", false);
```

`send()`方法接收一个参数，即要作为请求主体发送的数据。如果不需要通过请求主体发送数据，则必须传入 null，因为这个参数对有些浏览器来说是必需的。调用 `send()`之后，请求就会被分派到服务器
```
xhr.send(null);
```

收到响应后，响应 的数据会自动填充 XHR 对象的属性，相关的属性简介如下
```
// responseText:作为响应主体被返回的文本。
// responseXML:如果响应的内容类型是"text/xml"或"application/xml"，这个属性中将保存包含着响应数据的 XML DOM 文档。
// status:响应的 HTTP 状态。
// statusText:HTTP 状态的说明

xhr.open("get", "example.txt", false);
xhr.send(null);
if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
    alert(xhr.responseText);
} else {
    alert("Request was unsuccessful: " + xhr.status);
}
```
XHR 对象的 readyState 属性，该属性表示请求/响应过程的当前活动阶段
```
// 0:未初始化。尚未调用 open()方法。
// 1:启动。已经调用 open()方法，但尚未调用 send()方法。
// 2:发送。已经调用 send()方法，但尚未接收到响应。
// 3:接收。已经接收到部分响应数据。
// 4:完成。已经接收到全部响应数据，而且已经可以在客户端使用了。

var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function(){
  if (xhr.readyState == 4){
    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304){
      alert(xhr.responseText);
    } else {
      alert("Request was unsuccessful: " + xhr.status);
    }
  } 
};
xhr.open("get", "example.txt", true);
xhr.send(null);

```
#### HTTP头部信息
```
Accept:浏览器能够处理的内容类型。
Accept-Charset:浏览器能够显示的字符集。
Accept-Encoding:浏览器能够处理的压缩编码。
Accept-Language:浏览器当前设置的语言。
Connection:浏览器与服务器之间连接的类型。
Cookie:当前页面设置的任何 Cookie。
Host:发出请求的页面所在的域 。
Referer:发出请求的页面的 URI。注意，HTTP 规范将这个头部字段拼写错了，而为保证与规范一致，也只能将错就错了。(这个英文单词的正确拼法应该是 referrer。)
User-Agent:浏览器的用户代理字符串
```
要成功发送请求头部信息，必须在调用 `open()`方法之后且调用 `send()`方法 之前调用 `setRequestHeader()`
调用 XHR 对象的 `getResponseHeader()`方法并传入头部字段名称，可以取得相应的响应头部信息。而调用 `getAllResponseHeaders()`方法则可以取得一个包含所有头部信息的长字符串。

#### 跨域——同源策略(协议、域名、端口号不一致)限制
1、CORS(跨源资源共享)，使用自定义的HTTP头部让浏览器与服务器进行沟通，从而决定请求或相应是应该成功，还是失败。
2、JSONP(由回调函数和数据组成),缺点: 不安全, 难以确定请求是否成功, 只能发送get请求
JSONP 是通过动态`<script>`元素来使用的，使用时可以为src 属性指定一个跨域 URL。这里的`<script>`元素与`<img>`元素类似，都有能力不受限制地从其他域 加载资源。因为 JSONP 是有效的 JavaScript 代码，所以在请求完成后，即在 JSONP 响应加载到页面中 以后，就会立即执行
```
function handleResponse(response){
alert("You’re at IP address " + response.ip + ", which is in " +
  response.city + ", " + response.region_name);
}
var script = document.createElement("script");
script.src = "http://freegeoip.net/json/?callback=handleResponse"; 
document.body.appendChild(script);
```
#### Comet——服务器推送
1、长轮询，页面发起一个请求到服务器，然后服务器一直保持连接打开，直到有数据发送。发送完数据之后，浏览器关闭连接，随即又发起一个发到服务器的新请求(短轮询: 浏览器定时向服务器发送请求,看看有没有更新的数据)
2、HTTP流，浏览器向服务器发送一个请求，而服务器保持连接打开，然后周期性地向浏览器发送数据

#### Web Sockets——是在一个单独的持久连接上提供全双工、双向通信
由于 Web Sockets 使用了自定义的协议，所以 URL 模式也略有不同。未加密的连接不再是 http://， 而是 ws://;加密的连接也不是 https://，而是 wss://。在使用 Web Socket URL 时，必须带着这个 模式，因为将来还有可能支持其他模式。
优点: 能够在客户端和服务端之间发送非常少量的数据，而不必担心HTTP那样字节级的开销。由于传递的数据包很小，因此非常适合移动应用
缺点: 制定协议的时间比制定JavaScript API的时间还要长

#### 使用Web Sockets(不受同源策略限制)

1、实例化WebSocket对象并传入要连接的URL
```
var socket = new WebSocket("ws://www.example.com/server.php");
```
实例化了 WebSocket 对象后，浏览器就会马上尝试创建连接。与 XHR 类似，WebSocket 也有一个表示当前状态的 readyState 属性。不过，这个属性的值与 XHR 并不相同，而是如下所示。
WebSocket.OPENING (0):正在建立连接。 
WebSocket.OPEN (1):已经建立连接。 
WebSocket.CLOSING (2):正在关闭连接。 
WebSocket.CLOSE (3):已经关闭连接

2、发送和接收数据
```
var socket = new WebSocket("ws://www.example.com/server.php");
socket.send("Hello World!");
```
因为 Web Sockets 只能通过连接发送纯文本数据，所以对于复杂的数据结构，在通过连接发送之前，必须进行序列化，例如把数据序列化为一个 JSON 字符串，然后再发送到服务器
当服务器向客户端发来消息时，WebSocket 对象就会触发 message 事件。这个 message 事件与其他传递消息的协议类似，也是把返回的数据保存在 event.data 属性中
```
socekt.onmessage = function(e) {
  var data = e.data; // 返回的数据也是字符串
  // 处理数据
}
```
3、其他事件
WebSocket 对象还有其他三个事件，在连接生命周期的不同阶段触发。
open:在成功建立连接时触发。
error:在发生错误时触发，连接不能持续。
close:在连接关闭时触发。
