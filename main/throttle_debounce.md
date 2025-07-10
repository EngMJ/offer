> 节流和防抖经常在面试中被问到，也很容易搞混，这里就记录下节流和防抖函数的实现

## 1\. 节流

只在开始执行一次，未执行完成过程中触发的忽略，核心在于`开关锁`🔒。

例如：`多次点击按钮`提交表单，`第一次有效`

```js
// 节流
function throttle(fn, delay) {
    var timer = null;
    return function () {
        if (timer) { return false;}
        var that = this;
        var args = arguments;
        fn.apply(that, args);
        timer = setTimeout(function () {
            clearTimeout(timer);
            timer = null;
        }, delay || 500);
    };
}


// 使用
function clickHandler() {
    console.log('节流click!');
}
const handler = throttle(clickHandler);
document.getElementById('button').addEventListener('click', handler);
```

## 2\. 防抖

只执行最后一个被触发的，清除之前的异步任务，核心在于`清零`。

例如： `页面滚动`处理事件，搜索框输入联想  
`最后一次有效`

```js
// 防抖
function debounce(fn, delay) {
    var timer = null;
    return function () {
        var that = this;
        var args = arguments;
        clearTimeout(timer);// 清除重新计时
        timer = setTimeout(function () {
            fn.apply(that, args);
        }, delay || 500);
    };
}


// 使用
function clickHandler() {
    console.log('防抖click!');
}
const handler = debounce(clickHandler);
document.getElementById('button').addEventListener('click', handler);
```

### 图解对比:

**原始事件触发:**

| | | | | | | | | | | | | | | | | | | | | (连续快速触发事件)

**节流 (Throttle) - 每 100ms 执行一次:**

| _ _ _ _ | _ _ _ _ | _ _ _ _ | _ _ _ _ | (无论中间触发多少次，都按固定频率执行第一次,忽略后续触发)

**防抖 (Debounce) - 等待 100ms:**

_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ | (在最后一次触发后等待100ms，仅执行最后一次触发的事件)



关于容易搞混的问题，就记住：`节流是单位时间内只触发首次，防抖是只触发最后一次`。
