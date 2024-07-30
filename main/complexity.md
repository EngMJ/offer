通过图形方式帮助大家理解O(n2)、O(n3)和O(log⁡n)O(n^2)、O(n^3)和O(\\log n)，这样好处从本质理解这些计算函数复杂度的由来。

### O(n2)O(n^2)

```js
function square(arr){
    for (let i = 0; i < arr.length; i++) {
        for (let j = 0; j < arr.length; j++) {
            console.log(i,j)            
        }
        
    }
}
```

在上面的 square 函数中有两层嵌套的 for 循环，在循环内部执行`console.log(i,j)` 操作这个是一个常量时间复杂度，可以看成对矩阵的遍历，例如 `square(4)` 计算 `console.log(i,j)` 次数为 4×44 \\times 4 分别对应 i 和 j 取值，42\=164^2 = 16 所以其时间复杂度为 n2n^2

![屏幕快照 2021-10-24 下午1.49.19.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6c0849feb804ce39e8c0eb07cd751a2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

### O(n3)O(n^3)

```js
function cube(arr){
    for (let i = 0; i < arr.length; i++) {
        for (let j = 0; j < arr.length; j++) {
            for (let k = 0; k < arr.length; k++) {
                console.log(i,j,k)  
            }
        }
    }
}
```

可以将 3 层嵌套的 for 循环对应到立方体，假设 i 为列索引、j 为行索引和 k 为高索引，先沿 z(k) 轴生长，然后然 j 轴最后在将二维 j 和 k 组成的平面沿 i 轴方向堆叠得到立方体所以 4×4×4\=644 \\times 4 \\times 4 = 64 也就是 O(n3)O(n^3)

![cube_002.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c64870ee9dfe48ed953a8716316f34bb~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

### O(log⁡(n))O(\\log(n))

到现在来看都是n的某一个次方得到时间复杂度，那么如下图，多少的多少次方是 8，估计很快你就能给出答案 2 的 3 次方呗。如果要给出一般方法，就是对以 2 为 base 求 log。

```js
function logFunc(n){
    if(n===0) return "Done"
    n = Math.floor(n / 2);
    return logFunc(n)
}

```

在上面函数是一个调用自己的递归的函数，在层次函数对 n 进行除以 2 处理，退出条件为 `n===0` 我们通过图解列出其调用过程，移动调用自己 3 次，也就是这是 3 层深度的递归。输入 `n=8` 复杂度是3 如果用一个函数来表达他们之间的关系呢，可以log⁡n\\log n 也就是对 n 取以 2 为底对数就是函数的复杂度。

![log_003.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/001501df3f69430bba65cafc720c4a23~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)
