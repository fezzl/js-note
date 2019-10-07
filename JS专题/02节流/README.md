### 节流
节流的原理很简单：
如果你持续触发事件，每隔一段时间，只执行一次事件。

根据首次是否执行以及结束后是否执行，效果有所不同，实现的方式也有所不同。

我们用 leading 代表首次是否执行，trailing 代表结束后是否再执行一次。

关于节流的实现，有两种主流的实现方式，一种是使用时间戳，一种是设置定时器。
#### 使用时间戳
使用时间戳，当触发事件的时候，我们取出当前的时间戳，然后减去之前的时间戳(最一开始值设为 0 )，如果大于设置的时间周期，就执行函数，然后更新时间戳为当前的时间戳，如果小于，就不执行。
```
function throttle(func, wait) {
  let previous = 0;
  return function() {
    let context = this;
    let args = arguments;
    let now = +new Date();
    if(now - previous > wait) {
      func.apply(context, args);
      previous = now;
    }
  }
}
```
#### 使用定时器
当触发事件的时候，我们设置一个定时器，再触发事件的时候，如果定时器存在，就不执行，直到定时器执行，然后执行函数，清空定时器，这样就可以设置下个定时器。
```
function throttle(func, wait) {
  let timeout;
  return function() {
    let context = this;
    let args = arguments;
    if(!timeout) {
      timeout = function() {
        timeout = null;
        func.apply(context, args);
      }
    }
  }
}
```
所以比较两个方法：

第一种事件会立刻执行，第二种事件会在 n 秒后第一次执行
第一种事件停止触发后没有办法再执行事件，第二种事件停止触发后依然会再执行一次事件
#### 时间戳和定时器结合
想要一个有头有尾的！就是鼠标移入能立刻执行，停止触发的时候还能再执行一次
```
function throttle(func,wait) {
  let timeout,context,args;
  let previous = 0;
  let later = function() {
    previous = +new Date();
    timeout = null;
    func.apply(context,args);
  }
  let throttled = function() {
    let now = +new Date();
    context = this;
    args = arguments;
    let remaining = wait - (now - previous);
    if(remaining <= 0 || remaining > wait) {
      if(timeout) {
        clearTimeout(timeout);
        timeout = null;
      }
      previous = now;
      func.apply(context,args);
    } else if(!timeout) {
      timeout = setTimeout(later, remaining);
    }
  }
  return throttled
}
```
#### 优化
我们设置个 options 作为第三个参数，然后根据传的值判断到底哪种效果
leading：false 表示禁用第一次执行
trailing: false 表示禁用停止触发的回调
```
function throttle(func,wait,options) {
  let context,args,timeout;
  let previous = 0;
  if(!options) options = {};
  let later = function() {
    previous = options.leading === false ? 0 : new Date().getTime();
    timeout = null;
    func.apply(context,args);
  }
  let throttled = function() {
    let now = new Date().getTime();
    if(!previous && options.leading === false) previous = now;
    context = this;
    args = arguments;
    let remaining = wait - (now - previous);
    if(remaining <= 0 || remaining > wait) {
      if(timeout) {
        clearTimeout(timeout);
        timeout = null;
      }
      previous = now;
      func.apply(context,args);
      if(!timeout) context = args = null;
    } else if(!timeout && options.trailing !== false) {
      timeout = setTimeout(later, remaining);
    }
  }
  return throttled;
}
```