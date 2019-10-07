### 防抖
防抖的原理就是：你尽管触发事件，但是我一定在事件触发 n 秒后才执行，如果你在一个事件触发的 n 秒内又触发了这个事件，那我就以新的事件的时间为准，n 秒后才执行，总之，就是要等你触发完事件 n 秒内不再触发事件，我才执行

#### 第一版
```
function debounce(func,wait) {
  let timeout;
  return function() {
    clearTimeout(timeout);
    timeout = setTimeout(func,wait);
  }
}
```

#### this
如果是使用debounce函数的话this会指向window对象，所以需要修改this指向调用它的对象
```
function debounce(func,wait) {
  let timeout;
  return function() {
    let context = this;
    clearTimeout(timeout);
    timeout = setTimeout(function() {
      func.apply(context);
    },wait)
  }
}
```

#### event对象
js在事件处理函数中会提供event对象，如果使用了debounce函数的话就会打印undefined
所以，再次修改代码
```
function debounce(func,wait) {
  let timeout;
  return function() {
    let context = this;
    let args = arguments;
    clearTimeout(timeout);
    timeout = setTimeout(function() {
      func.apply(context,args);
    },wait)
  }
}
```
到此为止，我们解决了两个问题
- this指向
- event对象

#### 立即执行
如果我不希望非要等到事件停止触发后才执行，我希望立刻执行函数，然后等到停止触发 n 秒后，才可以重新触发执行
那我们加个immediate参数判断是否执行
```
function debounce(func,wait,immediate) {
  let timeout;
  return function() {
    let context = this;
    let args = arguments;
    if(timeout) clearTimeout(timeout);
    if(immediate) {
      let callNow = !timeout;
      timeout = setTimeout(function() {
        timeout = null;
      },wait)
      if(callNow) func.apply(context,args);
    } else {
      timeout = setTimeout(function() {
        func.apply(context,args);
      },wait)
    }
  }
}
```
#### 返回值
我们也要返回函数的执行结果，但是当 immediate 为 false 的时候，因为使用了 setTimeout ，我们将 func.apply(context, args)的返回值赋给变量，最后再 return 的时候，值将会一直是 undefined，所以我们只在 immediate 为 true 的时候返回函数的执行结果。
```
function debounce(func, wait, immediate) {
  let timeout, result;
  return function() {
    let context = this;
    let args = arguments;
    if(immediate) {
      let callNow = !timeout;
      timeout = setTimeout(function() {
        timeout = null;
      },wait)
      if(callNow) result = func.apply(context, args);
    } else {
      timeout = setTimeout(function() {
        func.apply(context, args);
      },wait)
    }
    return result;
  }
}
```
#### 取消
我希望能取消 debounce 函数，比如说我 debounce 的时间间隔是 10 秒钟，immediate 为 true，这样的话，我只有等 10 秒后才能重新触发事件，现在我希望有一个按钮，点击后，取消防抖，这样我再去触发，就可以又立刻执行啦
```
function debounce(func, wait, immediate) {
  let timeout, result;
  let debounced = function() {
    let context = this;
    let args = arguments;
    if(immediate) {
      let callNow = !timeout;
      timeout = setTimeout(function() {
        timeout = null;
      },wait)
      if(callNow) result = func.apply(context,args);
    } else {
      timeout = setTimeout(function() {
        func.apply(context,args);
      },wait)
    }
    return result;
  }
  debounced.cancel = function() {
    clearTimeout(timeout);
    timeout = null;
  }
  return debounced;
}
```