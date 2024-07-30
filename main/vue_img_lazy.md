## 什么是懒加载？

> 懒加载也叫做延迟加载、按需加载，指的是在长网页中延迟加载图片数据，是一种较好的网页性能优化的方式。在比较长的网页或应用中，如果图片很多，所有的图片都被加载出来，而用户只能看到可视窗口的那一部分图片数据，这样就浪费了性能。
>
> 如果使用图片的懒加载就可以解决以上问题。在滚动屏幕之前，可视化区域之外的图片不会进行加载，在图片进入可视区域后才加载。这样使得网页的加载速度更快，减少了服务器的负载。懒加载适用于图片较多，页面列表较长（长列表）的场景中。

## 懒加载带来的好处

+   **减轻服务器的压力**  
    对于一个展示大量图片的网页，使用懒加载可以显著地减少浏览器向服务器发送的请求数量，降低了服务器的压力，同时也减小了浏览器的负担。
+   **提升用户体验**  
    如果同时加载所有图片，会导致首屏的加载速度较慢，因为浏览器会把不在可视区域内的图片也一并加载。使用懒加载可以保证浏览器首先加载用户可视区域内的资源，提高用户体验。

## 懒加载的实现原理/如何实现懒加载

懒加载的核心原理主要是两块：

1.  **将图片的 src 属性置空，阻止图片加载：**  
    本文采用 [data-\*](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/data-* "https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/data-*") 自定义数据属性代替 src 来存储图片资源路径。笔者之前学习的时候也接触过将 src 备份后给 src 赋值为空字符串的方式，原理是一致的。
2.  **判断图片是否在可视区域内**  
    我们可以借助一些 API 来实现这一功能： [HTMLElement.offsetTop](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/offsetTop "https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/offsetTop")、[Element.scrollTop](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/scrollTop "https://developer.mozilla.org/zh-CN/docs/Web/API/Element/scrollTop")、[Element.clientHeight](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/clientHeight "https://developer.mozilla.org/zh-CN/docs/Web/API/Element/clientHeight")、[Window.innerHeight](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/innerHeight "https://developer.mozilla.org/zh-CN/docs/Web/API/Window/innerHeight")、[IntersectionObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver "https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver")，如果读者有不熟悉的 API，可以点击跳转到 mdn 浏览其文档，后面我们会详细介绍如何使用他们来判断图片是否在可视区域内。

懒加载的其他注意事项：

+   **触发方式**  
    假设我们判断图片是否在可视区域内并进行图片加载操作的函数是 `lazyLoad`，所谓触发方式，就是什么时候执行 `lazyLoad` 这个函数。传统的方式是监听 `window` 对象的 `scroll` 事件，即页面发生滚动时触发 `lazyLoad`。[IntersectionObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver "https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver") 这个 API 提供的功能则是对目标元素进行监听，但这是一个比较新的 api，要注意你的项目是否有浏览器兼容需求。它的回调函数触发条件也是被监听元素发生了滚动变化，这就为我们引入了下面两个问题。
+   **防抖/节流**  
    一次懒加载就可以加载一整个可视页面的图片，而滚动一个页面的距离会触发非常多次滚动事件，造成性能的浪费。针对这种场景，如果采用的是 `window.onscroll` 方式监听并触发懒加载，需要将懒加载函数套上一层防抖或者节流函数以限制其触发次数；如果采用的是 `IntersectionObserver`，它会逐个检测这些目标元素是否进入视窗，从而触发对应的加载。这就意味着不会一下子触发所有图片的加载，快速滑过时有的对象的回调函数还没来得及触发就已经不在可视区域内了。`IntersectionObserver` 会自动帮你管理这些，确保在性能和用户体验之间取得平衡。
+   ⭐**快速滑动至页面底部**⭐  
    大部分网上的代码在不使用 `IntersectionObserver` 时判断图片是否在可视区域内都仅仅考虑了从上向下滚动时，图片超过在页面底部，就触发懒加载了。那么假设这样一种场景：用户进入页面后快速滚动页面到底部，这样就会导致所有的图片都在可视区域出现过，进一步导致所有图片是否在可视区域内的判断条件都成立了，那么所有图片都会被加载，可能很久都不能加载出用户想看的底部图片。所以如果使用传统的计算方式判断图片是否可见时，既要计算是否超过可视区域底部，也要考虑是否已经滑过了到可视区域上边外部去了，同时配合节流限制懒加载函数的触发频率（这样快速滑动后懒加载被触发时，滑过但超出可视区域顶部的图片就不会被加载了）以达到性能优化。

## 懒加载的原生实现

首先我们编写一个 `html` 文件，后面用于展示懒加载的效果：

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Document</title>
        <style>
            img {
                height: 100vh;
                display: block;
                margin-bottom: 50px;
            }
        </style>
    </head>
    <body>
        <!-- 如果有占位图，可以将图片的src都设为占位图的路径 -->
        <img data-src="./img/1.jpg" alt="懒加载1" />
        <img data-src="./img/2.jpg" alt="懒加载2" />
        <img data-src="./img/3.jpg" alt="懒加载3" />
        <img data-src="./img/4.jpg" alt="懒加载4" />
        <img data-src="./img/5.jpg" alt="懒加载5" />
    </body>
    <script src="./lazyLoad.js"></script>
</html>
```

我们先把其中的 `data-src` 写成正常的 `src`，可以看到每次页面刷新时所有的图片都会被直接加载好。如果一个页面中有非常多的图片，这样同时加载所有图片对浏览器和服务器都会造成压力。

![GIF 2023-8-24 11-00-59.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c2fee1bda65492fa5f658d76740ff78~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

使用原生 JS 实现图片懒加载，有下面将呈现的三种方式。说是三种方式，其实就是借助了几种不同的 API，来实现判断当前图片是否在可视区域内。事实上这三种方式是随着原生 JS 推出了新的 API 而演变的，越新的 API 实现起来越便捷，而旧的 API 可以兼容老版本的浏览器。

### 方式一

第一种方式如下图，`图片顶部到文档顶部的距离 > 浏览器可视窗口高度 + 滚动条滚过的高度`，此时的图片就是不可见的，如果 `图片顶部到文档顶部的距离 < 浏览器可视窗口高度 + 滚动条滚过的高度` 那么该图片就应该出现在可视区域内了。  
但你还记得我们前面提到的注意事项吗？如果用户直接滑到页面底部，那么这个判断条件对所有的图片都为真，还是会造成性能问题。所以我们要再加上一条判断条件 `图片的高度 + 图片顶部到文档顶部的距离 > 滚动条滚过的高度`，以确保图片确实在可视区域内，而不只是被滑过。 ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7be83112f644b2a9a3bef0f85945040~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp) 当理解了这一判断条件之后，就只需要通过对应的 API 获取到他们即可：

+   待加载图片的高度：`img.clientHeight`
+   图片顶部到文档顶部的距离：`img.offsetTop`
+   浏览器窗口滚动过的距离：`document.documentElement.scrollTop` 或 `document.body.scrollTop`
+   浏览器可视窗口高度：`document.documentElement.clientHeight` 或 `window.innerHeight`


下面编写 js 代码：

```js
// 获取到所有的 img 标签对应的元素，存到 imgs 数组中
const imgs = document.getElementsByTagName('img');
function lazyLoad(imgs) {
  console.log('懒加载被触发了！')
  // 浏览器可视窗口的高度；
  const windowHeight = window.innerHeight;
  // 可视窗口滚动过的距离；
  const scrollHeight = document.documentElement.scrollTop;
  for (let i = 0; i < imgs.length; i++) {
    // 根据我们先前讲解过的是否可视逻辑进行判断；
    // !imgs[i].src 是当该图片已加载好之后，无需重复加载
    if (windowHeight + scrollHeight > imgs[i].offsetTop && !imgs[i].src && imgs[i].offsetTop + imgs[i].offsetHeight > scrollHeight) {
      // 使用data-xx的自定义属性可以通过dom元素的dataset.xx取得；
      imgs[i].src = imgs[i].dataset.src;
    }
  }
};
// 进入页面时执行一次加载；
lazyLoad(imgs);
// 监听滚动事件，进行图片懒加载；
window.onscroll = () => lazyLoad(imgs);
```

至此，我们就初步完成了一个图片懒加载功能的实现，让我们来看看效果：

![GIF 2023-8-24 14-58-51.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f621b7bec06e4796bbc15039f1b2e3c4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

还记得我们先前说过的通过监听滚动的方式实现懒加载需要进行防抖节流处理吗？我们可以看一下现在没有进行处理的代码，在 `lazyLoad` 函数中打一个 `console`，可以看到我们简单的五张图片来回滑动一下，滚动事件被触发了好几十次，大量的滚动事件对浏览器性能来说是一个很大的负担。

![GIF 2023-8-24 15-12-14.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d213664b5b8e48008fc1334421fc2ff5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

防抖和节流都可以规避频繁触发回调函数，懒加载应该和哪一种方式结合更好呢？笔者更加推荐采用节流的方式，因为防抖需要用户完全停止滚动一定时间才能进行加载，如果用户在查找指定图片而不松开控制滚动条的鼠标，很难保证完全不触发滚动事件，这样就会导致无法触发懒加载函数，图片展示不出来，带来的用户体验并不好。

添加节流函数对我们的懒加载功能进行优化：

```js
/**
 * 
 * @param {Function} fn 回调函数
 * @param {Number} delay 间隔时间
 * @param  {...any} args 回调函数 fn 需要用到的参数
 * @returns 
 */
function throttle(fn, delay, ...args) { // 使用剩余参数语法，接收任意数量的参数
  let timer = null; // 定义一个定时器
  return () => {
    let context = this; // 保存当前上下文
    if (!timer) { // 如果没有定时器
      timer = setTimeout(() => { // 设置一个延迟执行的定时器
        fn.apply(context, args); // 执行函数，并传入参数
        timer = null; // 清空定时器
      }, delay);
    }
  }
}
// 监听滚动事件，加载后面的图片；
window.onscroll = throttle(lazyLoad, 500, imgs);
```

❌**踩坑警告**❌：懒加载的场景不要使用时间戳实现的节流，这会导致当你快速滑到一个位置并立刻停止滑动时，无法进行图片的加载。

看一下添加节流函数后的效果：

![GIF 2023-8-24 15-54-59.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9edf5c9b3e054801bd7d40f411d6b590~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

同样的操作这次仅仅触发了几次懒加载函数，并且我们可以通过调整节流函数的参数，控制懒加载函数触发的频率，以达到平衡性能和用户体验的最佳效果。

### 方式二

[`Element.getBoundingClientRect()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect "https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect") 方法返回一个 [`DOMRect`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMRect "https://developer.mozilla.org/zh-CN/docs/Web/API/DOMRect") 对象，其提供了元素的大小及其相对于视口的位置。 如下图 ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a575097cc7d34addbc38b8277fe2d053~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp) 所以我们只需要判断 `元素相对于可视窗口顶部的距离 < 可视窗口高度` 来确保滚动条到了图片的位置，同时 `元素相对于可视窗口顶部的距离的绝对值 > 元素本身的高度` 来保证图片没有滚到可视窗口上方去。将方法一中的判断条件稍作更改，即得到了方法二：

```js
// 获取到所有的 img 标签对应的元素，存到 imgs 数组中
const imgs = document.getElementsByTagName('img');
function lazyLoad(imgs) {
  console.log('懒加载被触发了！')
  // 浏览器可视窗口的高度；
  const windowHeight = window.innerHeight;
  for (let i = 0; i < imgs.length; i++) {
    // 根据我们先前讲解过的是否可视逻辑进行判断；
    // !imgs[i].src 是当该图片已加载好之后，无需重复加载
    if (imgs[i].getBoundingClientRect().top < windowHeight && imgs[i].getBoundingClientRect().top > -imgs[i].clientHeight && !imgs[i].src) {
      // 使用data-xx的自定义属性可以通过dom元素的dataset.xx取得；
      imgs[i].src = imgs[i].dataset.src;
    }
  }
};

// 进入页面时执行一次加载；
lazyLoad(imgs);
// 监听滚动事件，加载后面的图片；
window.onscroll = throttle(lazyLoad, 500, imgs);
```

实现的效果、节流等部分与方法一中一致，这里就不再赘述了，主要区别只是使用了不同的 API。

### 方式三

[`IntersectionObserver`](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver "https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver") 是专门为检测某个元素是否出现在可视窗口中而推出的 API，我们这里简单介绍一下我们要用到的东西，对这个 API 感兴趣可以点击跳转到 mdn。

1.  **用法：**

```js
const observer = new IntersectionObserver(callback[, options]);
```

以 `IntersectionObserver` 构造函数新建一个对象，接收两个参数`callback`和`options`。

2.  **实例方法**

+   `observe()` 将一个元素加入监听目标集合
+   `unobserve()` 将一个元素移出监听目标集合

3.  **callback**  
    当监听目标发生滚动变化时触发的回调函数。该回调函数接受一个参数 `entries`, 它其实是 [`IntersectionObserverEntry`](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserverEntry "https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserverEntry") 的实例。简单来讲这个 `entries` 就存储着我们用 `observe()` 添加给 `observer` 实例的那些需要被监听的元素与其根元素容器在某一特定过渡时刻的交叉状态（默认为顶级文档的视窗）。而每一个 `entry` 有一个 `target` 属性，指向这个被监听的元素。

话不多说，我们直接上代码：

```js
document.addEventListener("DOMContentLoaded", () => {
  if ("IntersectionObserver" in window) {
    const imgs = document.getElementsByTagName("img");
    const imageObserve = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        // console.log("滚动触发监听函数了！")
        // 通过该属性判断元素是否出现在视口内
        if (entry.isIntersecting) {
          // entry.target能够取得那个dom元素
          const img = entry.target;
          img.src = img.dataset.src;
          // 图片加载完成后解除监听
          imageObserve.unobserve(img);
        }
      });
    });
    [...imgs].forEach((img) => {
      // 将所有的图片加入监听
      imageObserve.observe(img);
    });
  } else {
    alert("您的浏览器尚不支持IntersectionObserver，请尝试更新或者使用其他主流浏览器。");
  }
});
```

看看效果： ![GIF 2023-8-24 21-23-58.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4590a2aca2ae41bfaaa59b0a10ff4698~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2334&h=1179&e=gif&f=38&b=f6f4f6) 可以看到这一方法省去了繁琐的可视区域判断表达式，完美的实现了懒加载功能。别说用户快速下滑到页面底部，就是直接从底部开始也不会触发上面图片的加载了，非常的 nice。我们这里并没有进行节流处理，来看一下频繁滚动的场景会发生什么：

![GIF 2023-8-24 21-30-38.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a7715d138dd439eb79651fb66da15b8~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2334&h=1125&e=gif&f=37&b=fbf1f0) 打 `console` 的位置可以在上面代码的备注中看到，一开始的五次打印是将这五张图片加入监听，在我们向下滚动后图片都加载完成被移出了监听，来回滚动总共就触发了四次回调函数，而且整体的效果也比先前的计算判断条件丝滑很多。笔者也尝试了极速下滑至页面底部，中间的图片有时是不会进行加载的，可见该 API 确实对性能和用户体验进行了很好的平衡。  
美中不足的是，mdn 中该 API 的许多页面都标注着实验性技术，如果你的项目有兼容性需求，先检查一下是否可用。

## vue 实现懒加载指令 v-lazy

[`自定义指令`](https://cn.vuejs.org/guide/reusability/custom-directives.html#hook-arguments "https://cn.vuejs.org/guide/reusability/custom-directives.html#hook-arguments") 类似于我们常常在模板中使用的 `v-if`，`v-show` 等指令，可以通过 `app.directive` 来全局注册。 我们可以在 `src` 目录下新建一个 `directives` 文件夹，存放我们自定义的指令。

```js
const lazyLoad = {
  // mounted 在绑定元素的父组件 
  // 及他自己的所有子节点都挂载完成后调用
  mounted(el, binding) {
    // 如果有需要可以先设置src为 loading 图 
    // el.setAttribute('src', 'loading 图的路径');
    const options = {
      rootMargin: '0px',
      threshold: 0.1,
    };
    const observer = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          // binding 是一个对象，binding.value 是传递给指令的值
          el.setAttribute('src', binding.value);
          observer.unobserve(el);
        }
      });
    }, options);
    observer.observe(el);
  },
};

export default lazyLoad;
```

在 `main.js` 里添加注册我们的 `v-lazy` 指令：

```js
import lazyLoad from './directives/lazy'

const app = createApp(App)
app.directive('lazy', lazyLoad)
```

最后在组件中使用该指令：

```js
<template>
  <div>
    <img v-lazy="'src/views/lazyLoad/img/1.jpg'" alt="懒加载1" />
    <img v-lazy="'src/views/lazyLoad/img/2.jpg'" alt="懒加载2" />
    <img v-lazy="'src/views/lazyLoad/img/3.jpg'" alt="懒加载3" />
    <img v-lazy="'src/views/lazyLoad/img/4.jpg'" alt="懒加载4" />
    <img v-lazy="'src/views/lazyLoad/img/5.jpg'" alt="懒加载5" />
  </div>
</template>

<script setup></script>

<style lang="scss" scoped>
img {
  height: 100vh;
  display: block;
  margin-bottom: 50px;
}
</style>
```

这种方式是为了方便快捷的使用懒加载，并且可以自己选择对哪些图片进行懒加载。至于具体的实现，原生中的每一种实现方式在这里都是可以用的，只不过多套了一层自定义指令的壳。

⭐**注意事项**⭐

自定义指令是在 `main.js` 中引入的，不会被打包工具编译，只有 `src` 目录下的文件才会被编译。故而在 'lazy.js' 中是不能够使用相对路径的。解决方案：

+   使用绝对路径
+   使用 `requie` 引入路径（Webpack）/ 使用 `getImageUrl` 引入路径（Vite）

```js
<template>
  <div>
    <img v-lazy="getImageUrl('1.jpg')" alt="懒加载1" />
    <img v-lazy="getImageUrl('2.jpg')" alt="懒加载2" />
    <img v-lazy="getImageUrl('3.jpg')" alt="懒加载3" />
    <img v-lazy="getImageUrl('4.jpg')" alt="懒加载4" />
    <img v-lazy="getImageUrl('5.jpg')" alt="懒加载5" />
  </div>
</template>

<script setup>
const getImageUrl = (name) => {
  return new URL(`./img/${name}`, import.meta.url).href
}
</script>
```

笔者这里使用的是 `Vite`，我们稍稍修改用以展示的组件。其实对于本地图片场景，绝对路径也挺方便；如果图片都是在线链接那就最好，无需处理。笔者只想到一种情况需要通过打包工具的静态资源处理来加载图片，那就是图片是存在本地的，选择哪张图片又和数据有关。遇到这样的场景我们只需要把 `getImageUrl` 的参数修改成对应的变量即可。

但这样写其实不太方便，总要定义 `getImageUrl` 这个函数，要么写到工具方法中，要么每个需要引入静态资源的组件都要写一遍。所以可以考虑使用 [vite-plugin-require-transform](https://www.npmjs.com/package/vite-plugin-require-transform "https://www.npmjs.com/package/vite-plugin-require-transform") 这样的插件来解决此问题。
