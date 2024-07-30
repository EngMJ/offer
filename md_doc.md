![Markdown 的全方位应用指南](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a8f2956d0f04479abba11288c733e97~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2161&h=2136&s=289932&e=png&b=2d2d30)

Markdown 是一种轻量级标记语言，设计初衷是使写作和编辑文档变得简单而直观。它通过使用易于阅读和书写的纯文本格式，同时允许添加格式化元素，如标题、列表、链接等，来代替复杂的富文本编辑器。Markdown 的主要优点在于其简洁性和易学性，使得用户可以专注于内容而非格式。

主要特点和用途：

+   **简洁性和易读性**：Markdown 的语法设计简单明了，使用起来像是书写普通文本，但能够快速转换成结构化文档。

+   **跨平台兼容**：Markdown 文档可以在几乎所有操作系统上使用，并且与许多应用程序和平台兼容。

+   **易于学习和使用**：不需要复杂的培训或专业知识，即可快速上手。

+   **适用于多种用途**：Markdown 可用于撰写博客文章、文档、笔记、电子书等多种应用场景。

+   **版本控制友好**：由于是纯文本格式，Markdown 文档易于进行版本控制，如使用 Git 等工具管理和比较文本的变化。

+   **支持 HTML 嵌入**：Markdown 允许嵌入原生 HTML 代码，因此也适用于需要更高级格式化或交互性的内容。


## Markdown 基础

### 标题

使用 `#` 号可表示 1-6 级标题，一级标题对应一个 `#` 号，二级标题对应两个 `#` 号，以此类推。例如：

```shell
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3dfad2335ab14e8485ef55ffbedd597f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=474&h=514&s=37658&e=png&b=1f1f1f)

### 段落和换行

段落之间用空行分隔，Markdown 会自动识别为不同段落。若需要强制换行，可在行尾添加两个以上的空格，然后回车。

### 字体及颜色

#### 字体

Markdown 可以使用以下几种字体：

```markdown
*斜体文本*
_斜体文本_
**粗体文本**
__粗体文本__
***粗斜体文本***
___粗斜体文本___
```

效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6477d8d78cb841feaaf7bd935567e57d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=386&h=328&s=31404&e=png&b=1e1e1e)

#### 字体颜色

```xml
<font size=1>大小为1的字体</font>
<font size=6>大小为6的字体</font>

<font color=gray size=4>gray颜色的字</font>
<font color=green size=4>green颜色的字</font>
<font color=hotpink size=4>hotpink颜色的字</font>
<font color=LightCoral size=4>LightCoral颜色的字</font>
<font color=LightSlateGray size=4>LightSlateGray颜色的字</font>
<font color=orangered size=4>orangered颜色的字</font>
<font color=red size=4>red颜色的字</font>
<font color=springgreen size=4>springgreen颜色的字</font>
<font color=Turquoise size=4>Turquoise颜色的字</font>
```

效果如下： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe1ce380da394e83979f2ffc426d4e5e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1726&h=182&s=86369&e=png&b=1e1e1e)

### 分割线

```markdown
***
* * *
******
- - -
_ _ _
```

效果如下（显示的黄色线条是因为我的typora主题所致）： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c32990198494acb8877f17e4c55e5ec~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1726&h=286&s=12674&e=png&b=1e1e1e)

```markdown
***
分割线中间的内容
* * *
分割线中间的内容
******
分割线中间的内容
- - -
分割线中间的内容
_ _ _
```

效果图如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/209af495bd9d429f9c9c0bd482e71ee6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1724&h=662&s=55606&e=png&b=1e1e1e)

### 下划线

Markdown中并无下划线的原生语法，可以通过HTML的 `<u>` 标签来实现。

```xml
<u>我带下划线</u>
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c4149fd59a942398870b8d7857775dc~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=574&h=116&s=9484&e=png&b=1e1e1e)

### 删除线

删除线是用两个 `~~` 包裹内容。

```makefile
原价: ~~￥10.00~~
```

效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b11689a9525340d5b442e6874431684f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=524&h=90&s=8468&e=png&b=1e1e1e)

### 列表

Markdown 支持有序列表和无序列表。无序列表使用星号(`*`)、加号(`+`)或是减号(`-`)作为列表标记，这些标记后面要添加一个空格，然后再填写内容：

无序列表使用 `*`、`-` 或 `+` 开头：

```markdown
- 项目一
- 项目二
  * 子项目
  * 子项目
+ 项目三
+ 项目四
```

效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9c0764e780d4bb7828b73af787ecd45~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=468&h=416&s=17511&e=png&b=1e1e1e)

有序列表使用数字后加英文句点：

```markdown
1. 第一步
2. 第二步
   1. 子步骤
   2. 子步骤
4. 强调和粗体
```

列表嵌套只需在子列表中的选项前面添加两个或四个空格即可：

```markdown
1. 第一项：
    - 第一项嵌套的第一个元素
    - 第一项嵌套的第二个元素
2. 第二项：
    - 第二项嵌套的第一个元素
    - 第二项嵌套的第二个元素
```

效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ac87e5c0f594c3a9eb03859a65b02b9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=660&h=396&s=47171&e=png&b=1e1e1e)

### 图片

```scss
![alt 属性文本](图片地址)
![alt 属性文本](图片地址 "可选标题")
```

+   开头一个感叹号 !
+   接着一个方括号，里面放上图片的替代文字
+   接着一个普通括号，里面放上图片的网址，最后还可以用引号包住并加上选择性的 'title' 属性的文字。

```less
![image](https://files.mdnice.com/user/8213/898da997-7e84-4466-ae76-2628dcf7a2c3.png)
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e038eba3bf264c53a80d0bea66d6abfa~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1400&h=791&s=133618&e=png&b=ffffff)

还可以使用 HTML 的方式引入图片：

```ini
<img src="https://files.mdnice.com/user/8213/898da997-7e84-4466-ae76-2628dcf7a2c3.png" style="width: 20em" alt="HTML show time" />
```

![HTML show time](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60adaf1fb76d473fa56777dbc3b55227~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1400&h=791&s=133618&e=png&b=ffffff)

### 链接

```scss
[链接名称](链接地址)

或者

<链接地址>
```

效果图如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cdb1864986ff416499bac0fa50bfa567~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=296&h=246&s=15627&e=png&b=1e1e1e)

#### 高级链接

我们可以通过变量来设置一个链接，变量赋值在文档末尾进行：

```less
这个链接用 1 作为网址变量 [Google][1]
这个链接用 2 作为网址变量 [MDN][2]
然后在文档的结尾为变量赋值（网址）

[1]: http://www.google.com/
[2]: https://developer.mozilla.org/en-US/
```

效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d5a8f34394a4140b171df4a08ca3641~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=678&h=326&s=58936&e=png&b=1e1e1e)

### 引用

使用 `>` 表示引用，每一层嵌套加上 `>` 符号和空格

```markdown
> 区块
> > 嵌套1
> > > 嵌套2
```

效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d689aa15b7b41ed86ce45c6ff877078~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1576&h=484&s=25938&e=png&b=f9f9f9)

### 代码块和语法高亮

当行代码块用单个单引号包裹代码，代码块使用三个反引号 \`\`\` 包围，可以指定语言进行语法高亮：

+   单行代码块 在 JavaScript 中可以通过 `window.open("url")` 打开一个新窗口。 效果如下： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a59a7c8414d49549255577ed28d5220~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=974&h=92&s=21223&e=png&b=202020)

+   多行代码块

    ```go
    package main
    
    import (
        "fmt"
    
        "github.com/gogf/gf/v2/frame/g"
        "github.com/gogf/gf/v2/os/gcache"
        "github.com/gogf/gf/v2/os/gctx"
    )
    
    func main() {
        c := gcache.New()
        ctx := gctx.New()
    
        // 使用 SetMap 方法向缓存中设置多个键值对，键 "k1" 对应的值为 "v1"，过期时间为 0 表示永不过期
        c.SetMap(ctx, g.MapAnyAny{"k1": "v1"}, 0)
    
        // 使用 Data 方法获取整个缓存的数据
        data, _ := c.Data(ctx)
        fmt.Println(data) // map[k1:v1]
    }
    ```

    效果如下(具体效果与markdown的主题有关系)： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc6299cb2cb6452d89424c49ce998ac7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1698&h=1110&s=159587&e=png&b=292929)


### 转义字符

使用反斜杠 `\` 可以转义 Markdown 的特殊符号，如 `\*` 表示星号而不是斜体符号。

```markdown
**文本加粗** 
\*\* 正常显示星号 \*\*
```

效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f75d630acb854cc9bafc71cb67ec9ed7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=360&h=136&s=11377&e=png&b=1e1e1e)

Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号：

```markdown
\   反斜线
`   反引号
*   星号
_   下划线
{}  花括号
[]  方括号
()  小括号
#   井字号
+   加号
-   减号
.   英文句点
!   感叹号
```

这些是 Markdown 的基础语法元素，能够满足大多数常见的文本格式化需求。通过这些简单的符号和约定，用户可以轻松地将纯文本转换成结构化和格式化的文档。

### 脚注

Markdown 的扩展语法中支持脚注，可以在文档中添加参考注释。这在撰写学术文档时特别有用。

```markdown
这是一个需要脚注的句子[^1]。

[^1]: 这是脚注的内容。
```

效果图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49c8146794514268a73810cd31c2ab5a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=498&h=178&s=20293&e=png&b=1f1f1f)

### 表格

标准 Markdown 中，表格支持使用简单的管道符 `|` 和破折号 `-` 来定义：

```lua
| 标题1 | 标题2 | 标题3 |
|-------|-------|-------|
| 内容1 | 内容2 | 内容3 |
| 内容4 | 内容5 | 内容6 |
```

效果图如下： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/275e8d441ea341459e4b0f6c42cf343d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1726&h=284&s=30844&e=png&b=202020)

### 任务列表

[GFM](https://github.github.com/gfm/ "https://github.github.com/gfm/") 支持任务列表，可以用于创建带有复选框的任务列表：

```css
- [x] 完成的任务
- [ ] 未完成的任务
```

效果图如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a284f477dcf54d82b14756150bedd699~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=548&h=148&s=16088&e=png&b=1e1e1e)

## Markdown 进阶用法

### emoji 符号

Markdown 支持一些表情符号的使用，例如 :smile: 被渲染为 😄。

支持的 Emoji 符号表格（包含但不限于以下内容）：

| Emoji 符号 | 描述 | Markdown 语法 | 渲染结果 |
| --- | --- | --- | --- |
| :smile: | 笑脸 | `:smile:` | 😄 |
| :heart: | 红心 | `:heart:` | ❤️ |
| :+1: | 点赞 | `:+1:` | 👍 |
| :-1: | 踩 | `:-1:` | 👎 |
| :pray: | 祈祷 | `:pray:` | 🙏 |
| :fire: | 火焰 | `:fire:` | 🔥 |
| :rocket: | 火箭 | `:rocket:` | 🚀 |
| :star: | 星星 | `:star:` | ⭐ |
| :zap: | 闪电 | `:zap:` | ⚡ |
| :sunny: | 太阳 | `:sunny:` | ☀️ |
| :rainbow: | 彩虹 | `:rainbow:` | 🌈 |
| :umbrella: | 雨伞 | `:umbrella:` | ☔ |
| :bulb: | 灯泡 | `:bulb:` | 💡 |
| :moneybag: | 钱袋 | `:moneybag:` | 💰 |
| :hourglass: | 沙漏 | `:hourglass:` | ⌛ |
| :calendar: | 日历 | `:calendar:` | 📅 |
| :computer: | 计算机 | `:computer:` | 💻 |
| :iphone: | 手机 | `:iphone:` | 📱 |
| :email: | 电子邮件 | `:email:` | 📧 |
| :camera: | 相机 | `:camera:` | 📷 |
| :video\_camera: | 视频相机 | `:video_camera:` | 📹 |
| :tv: | 电视 | `:tv:` | 📺 |
| :loudspeaker: | 扬声器 | `:loudspeaker:` | 🔊 |
| :bell: | 钟铃 | `:bell:` | 🔔 |
| :gift: | 礼物盒 | `:gift:` | 🎁 |
| :balloon: | 气球 | `:balloon:` | 🎈 |
| :cake: | 蛋糕 | `:cake:` | 🍰 |
| :coffee: | 咖啡 | `:coffee:` | ☕ |
| :beer: | 啤酒 | `:beer:` | 🍺 |
| :cocktail: | 鸡尾酒 | `:cocktail:` | 🍸 |
| :football: | 足球 | `:football:` | ⚽ |
| :basketball: | 篮球 | `:basketball:` | 🏀 |
| :guitar: | 吉他 | `:guitar:` | 🎸 |
| :microphone: | 麦克风 | `:microphone:` | 🎤 |
| :headphones: | 耳机 | `:headphones:` | 🎧 |
| :rainbow\_flag: | 彩虹旗 | `:rainbow_flag:` | 🏳️‍🌈 |
| :checkered\_flag: | 方格旗 | `:checkered_flag:` | 🏁 |
| :shield: | 盾牌 | `:shield:` | 🛡️ |
| :star\_and\_crescent: | 星月标志 | `:star_and_crescent:` | ☪️ |
| :crossed\_swords: | 交叉剑 | `:crossed_swords:` | ⚔️ |
| :medal: | 奖牌 | `:medal:` | 🎖️ |
| :trophy: | 奖杯 | `:trophy:` | 🏆 |
| :gift\_heart: | 心形礼物 | `:gift_heart:` | 💝 |
| :sparkles: | 闪耀 | `:sparkles:` | ✨ |
| :boom: | 爆炸 | `:boom:` | 💥 |
| :sos: | 求救信号 | `:sos:` | 🆘 |
| :warning: | 警告 | `:warning:` | ⚠️ |
| :question: | 问号 | `:question:` | ❓ |
| :exclamation: | 惊叹号 | `:exclamation:` | ❗ |
| :bangbang: | 双感叹号 | `:bangbang:` | ‼️ |
| :heavy\_check\_mark: | 大勾号 | `:heavy_check_mark:` | ✔️ |
| :x: | 大叉号 | `:x:` | ❌ |
| :wavy\_dash: | 波浪线 | `:wavy_dash:` | 〰️ |
| :ok\_hand: | OK 手势 | `:ok_hand:` | 👌 |
| :thumbsup: | 点赞手势 | `:thumbsup:` | 👍 |
| :clap: | 鼓掌 | `:clap:` | 👏 |
| :heart\_eyes: | 心目中的眼睛 | `:heart_eyes:` | 😍 |
| :facepunch: | 拳头 | `:facepunch:` | 👊 |
| :v: | V 手势 | `:v:` | ✌️ |
| :metal: | 手势 | `:metal:` | 🤘 |
| :sweat\_smile: | 汗笑脸 | `:sweat_smile:` | 😅 |
| :tada: | 庆祝 | `:tada:` | 🎉 |
| :100: | 百分之百 | `:100:` | 💯 |
| :fireworks: | 烟花 | `:fireworks:` | 🎆 |
| :sparkler: | 火花棒 | `:sparkler:` | 🎇 |
| :moon: | 月亮 | `:moon:` | 🌜 |
| :sun\_with\_face: | 太阳与脸 | `:sun_with_face:` | 🌞 |
| :alien: | 外星人 | `:alien:` | 👽 |
| :ghost: | 幽灵 | `:ghost:` | 👻 |
| :skull: | 骷髅头 | `:skull:` | 💀 |
| :robot: | 机器人 | `:robot:` | 🤖 |

### 目录和跳转

Markdown 支持通过链接和引用实现文档内的跳转。这对于长文档非常有用，可以在页面顶部添加目录链接到各个部分。

```ini
## 目录
- [简介](#简介)
- [基础语法](#基础语法)
- [高级用法](#高级用法)

## 简介
...

## 基础语法
...

## 高级用法
...
```

展示效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a194b9a609b84c23815b7006c266960a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=512&h=784&s=37487&e=png&b=1e1e1e)

当点击“简介”、“基础语法”、“高级用法”等链接时，会跳转到对应的部分，相当于 HTML 中 a 标签的锚链接。

### LaTeX 公式

Markdown 支持使用 LaTeX 语法来编写数学公式，特别适合需要在文档中展示数学表达式的情况。公式可以有行内和块级两种形式：

+   行内公式用 `$...$` 包围，例如：

    ```ini
    这是一个行内公式：$E = mc^2$
    ```

    效果图如下：

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ba1eb94ce3d4456af632ca9012389ec~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=494&h=122&s=12759&e=png&b=1e1e1e)
+   块级公式用 `$$...$$` 包围，例如：

    ```css
    $$
    \int_{a}^{b} f(x) \, dx
    $$
    ```

    效果图如下：

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3985e4c9aef543a7b960578b552ea272~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=640&h=214&s=12271&e=png&b=1e1e1e)

### 自定义样式和主题

Markdown 本身不支持复杂的样式，但可以通过 HTML 以及与支持 CSS 的渲染工具结合来实现。例如，可以在 Markdown 中直接插入 HTML 代码来指定样式：

```css
<div style="color: red; font-size: 18px;">
这是自定义样式的文本。
</div>
```

效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b02401885fa1445b82592b49d836c269~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=520&h=108&s=12697&e=png&b=1e1e1e)

一些 Markdown 渲染器还支持自定义主题，通过修改 CSS 可以改变整个文档的外观。

### 嵌入媒体

除了图片，Markdown 也可以嵌入其他类型的媒体，如视频和音乐。虽然 Markdown 本身不直接支持这些功能，但可以通过 HTML 嵌入。

#### 嵌入视频

```css
<audio controls>
  <source src="path/to/audio/file.mp3" type="audio/mpeg">
  你的浏览器不支持音频元素。
</audio>
```

#### 嵌入音乐

```css
<audio controls>
  <source src="path/to/audio/file.mp3" type="audio/mpeg">
  你的浏览器不支持音频元素。
</audio>
```

## Markdown 高级用法

### 数学公式

#### LaTeX 公式基础介绍

在 Markdown 文档中，可以使用 LaTeX 语法来编写数学公式，尤其适用于需要在文档中展示数学表达式的情况。Markdown 本身不支持直接的数学公式，但许多 Markdown 渲染器和工具（如 GitHub、Jupyter Notebook、Markdown 预览工具等）都内置或支持 LaTeX 公式渲染。

#### 常用的数学符号

+   **希腊字母**

    | 字母名称 | 小写希腊字母 | LaTeX 小写语法 | 小写渲染效果 | 大写希腊字母 | LaTeX 大写语法 | 大写渲染效果 | 用途示例（数学/物理） |
        | --- | --- | --- | --- | --- | --- | --- | --- |
    | Alpha | α | `\alpha` | α\\alpha | Α | `A` | AA | α：角度, 误差; Α：吸收系数 |
    | Beta | β | `\beta` | β\\beta | Β | `B` | BB | β：β射线, 衰减因子; Β：磁场强度 |
    | Gamma | γ | `\gamma` | γ\\gamma | Γ | `\Gamma` | Γ\\Gamma | γ：伽马射线, 洛伦兹因子; Γ：Gamma函数 |
    | Delta | δ | `\delta` | δ\\delta | Δ | `\Delta` | Δ\\Delta | δ：变分, 不确定性; Δ：变化量 |
    | Epsilon | ε | `\epsilon` | ϵ\\epsilon | Ε | `E` | EE | ε：小量, 介电常数; Ε：电场强度 |
    | Zeta | ζ | `\zeta` | ζ\\zeta | Ζ | `Z` | ZZ | ζ：阻尼比; Ζ：Zeta函数 |
    | Eta | η | `\eta` | η\\eta | Η | `H` | HH | η：效率, 黏性; Η：磁场 |
    | Theta | θ | `\theta` | θ\\theta | Θ | `\Theta` | Θ\\Theta | θ：角度, 温度; Θ：德拜温度 |
    | Iota | ι | `\iota` | ι\\iota | Ι | `I` | II | ι：小角度; Ι：电流强度 |
    | Kappa | κ | `\kappa` | κ\\kappa | Κ | `K` | KK | κ：曲率, 导热系数; Κ：非线性系数 |
    | Lambda | λ | `\lambda` | λ\\lambda | Λ | `\Lambda` | Λ\\Lambda | λ：波长, 特征值; Λ：宇宙常数 |
    | Mu | μ | `\mu` | μ\\mu | Μ | `M` | MM | μ：磁导率, 摩擦系数; Μ：质量, 矩阵 |
    | Nu | ν | `\nu` | ν\\nu | Ν | `N` | NN | ν：频率, 泊松比; Ν：粒子数 |
    | Xi | ξ | `\xi` | ξ\\xi | Ξ | `\Xi` | Ξ\\Xi | ξ：随机变量, 位置; Ξ：Ξ粒子 |
    | Omicron | ο | `o` | oo | Ο | `O` | OO | ο：无具体用途; Ο：无具体用途 |
    | Pi | π | `\pi` | π\\pi | Π | `\Pi` | Π\\Pi | π：圆周率, 流体压力; Π：乘积运算符 |
    | Rho | ρ | `\rho` | ρ\\rho | Ρ | `P` | PP | ρ：密度, 电阻率; Ρ：反射率 |
    | Sigma | σ (ς) | `\sigma` (终结) | σ\\sigma (终结) | Σ | `\Sigma` | Σ\\Sigma | σ：标准偏差, 应力; Σ：求和运算符 |
    | Tau | τ | `\tau` | τ\\tau | Τ | `T` | TT | τ：时间常数, 切应力; Τ：温度 |
    | Upsilon | υ | `\upsilon` | υ\\upsilon | Υ | `\Upsilon` | Υ\\Upsilon | υ：中微子, 速度; Υ：宇宙学符号 |
    | Phi | φ (ϕ) | `\phi` (alt) | ϕ\\phi (alt) | Φ | `\Phi` | Φ\\Phi | φ：角, 电势; Φ：磁通量 |
    | Chi | χ | `\chi` | χ\\chi | Χ | `X` | XX | χ：磁化率, 敏感度; Χ：统计学, 高阶符号 |
    | Psi | ψ | `\psi` | ψ\\psi | Ψ | `\Psi` | Ψ\\Psi | ψ：波函数, 流体势; Ψ：希尔伯特空间 |
    | Omega | ω | `\omega` | ω\\omega | Ω | `\Omega` | Ω\\Omega | ω：角速度, 频率; Ω：欧姆, 电阻 |

+   **运算符和符号**

    | 符号名称 | 符号 | LaTeX 语法 | 渲染效果 | 用途示例 |
        | --- | --- | --- | --- | --- |
    | 加法 | + | `+` | ++ | 加法运算，例如 a+ba + b |
    | 减法 | \- | `-` | −\- | 减法运算，例如 a−ba - b |
    | 乘法（点积） | · | `\cdot` | ⋅\\cdot | 点积，例如 a⋅ba \\cdot b |
    | 乘法（叉积） | × | `\times` | ×\\times | 叉积，例如 a×ba \\times b |
    | 除法 | ÷ | `\div` | ÷\\div | 除法运算，例如 a÷ba \\div b |
    | 等于 | \= | `=` | \=\= | 等于，例如 a\=ba = b |
    | 不等于 | ≠ | `\neq` | ≠\\neq | 不等于，例如 a≠ba \\neq b |
    | 大于 | `>` | \>\> | 大于，例如 a\>ba > b |
    | 小于 | < | `<` | << | 小于，例如 a<ba < b |
    | 大于等于 | ≥ | `\geq` | ≥\\geq | 大于等于，例如 a≥ba \\geq b |
    | 小于等于 | ≤ | `\leq` | ≤\\leq | 小于等于，例如 a≤ba \\leq b |
    | 近似等于 | ≈ | `\approx` | ≈\\approx | 近似等于，例如 a≈ba \\approx b |
    | 恒等于 | ≡ | `\equiv` | ≡\\equiv | 恒等于，例如 a≡ba \\equiv b |
    | 比例 | ∝ | `\propto` | ∝\\propto | 成比例，例如 a∝ba \\propto b |
    | 加减 | ± | `\pm` | ±\\pm | 加减，例如 a±ba \\pm b |
    | 乘积 | ∏ | `\prod` | ∏\\prod | 乘积运算，例如 ∏i\=1nai\\prod\_{i=1}^n a\_i |
    | 求和 | ∑ | `\sum` | ∑\\sum | 求和运算，例如 ∑i\=1nai\\sum\_{i=1}^n a\_i |
    | 交集 | ∩ | `\cap` | ∩\\cap | 集合交集，例如 A∩BA \\cap B |
    | 并集 | ∪ | `\cup` | ∪\\cup | 集合并集，例如 A∪BA \\cup B |
    | 空集 | ∅ | `\emptyset` | ∅\\emptyset | 空集，例如 A\=∅A = \\emptyset |
    | 子集 | ⊆ | `\subseteq` | ⊆\\subseteq | 子集，例如 A⊆BA \\subseteq B |
    | 真子集 | ⊂ | `\subset` | ⊂\\subset | 真子集，例如 A⊂BA \\subset B |
    | 超集 | ⊇ | `\supseteq` | ⊇\\supseteq | 超集，例如 A⊇BA \\supseteq B |
    | 真超集 | ⊃ | `\supset` | $\\supset\` | 真超集，例如 A⊃BA \\supset B |
    | 并非子集 | ⊄ | `\nsubseteq` | ⊈\\nsubseteq | 并非子集，例如 A⊈BA \\nsubseteq B |
    | 任意 | ∀ | `\forall` | ∀\\forall | 对于所有，例如 ∀x∈X\\forall x \\in X |
    | 存在 | ∃ | `\exists` | ∃\\exists | 存在，例如 ∃x∈X\\exists x \\in X |
    | 归属 | ∈ | `\in` | ∈\\in | 属于，例如 x∈Ax \\in A |
    | 不归属 | ∉ | `\notin` | ∉\\notin | 不属于，例如 x∉Ax \\notin A |
    | 平方 | ^2 | `^{2}` | 2^{2} | 平方，例如 a2a^{2} |
    | 立方 | ^3 | `^{3}` | 3^{3} | 立方，例如 a3a^{3} |
    | 幂 | ^n | `^{n}` | n^{n} | 幂，例如 ana^{n} |
    | 平方根 | √ | `\sqrt` | \\sqrt{} | 平方根，例如 a\\sqrt{a} |
    | n次方根 | ∛ | `\sqrt[n]` | n\\sqrt\[n\]{} | n次方根，例如 a3\\sqrt\[3\]{a} |
    | 自然对数 | ln | `\ln` | ln⁡\\ln | 自然对数，例如 ln⁡(a)\\ln(a) |
    | 对数 | log | `\log` | log⁡\\log | 对数，例如 log⁡(a)\\log(a) |
    | 阶乘 | ! | `!` | !! | 阶乘，例如 n!n! |
    | 分数 | / | `\frac` | ab\\frac{a}{b} | 分数，例如 ab\\frac{a}{b} |
    | 极限 | lim | `\lim` | lim⁡\\lim | 极限，例如 lim⁡x→∞f(x)\\lim\_{x \\to \\infty} f(x) |
    | 极小值 | min | `\min` | min⁡\\min | 极小值，例如 min⁡(a,b)\\min(a, b) |
    | 极大值 | max | `\max` | max⁡\\max | 极大值，例如 max⁡(a,b)\\max(a, b) |
    | 上标 |  | `^{}` | ^{ } | 上标，例如 aba^{b} |
    | 下标 |  | `_` | \_{} | 下标，例如 aba\_{b} |
    | 导数 | ' | `'` | ′' | 导数，例如 f′(x)f'(x) |
    | 偏导数 | ∂ | `\partial` | ∂\\partial | 偏导数，例如 ∂f∂x\\frac{\\partial f}{\\partial x} |
    | 无穷大 | ∞ | `\infty` | ∞\\infty | 无穷大，例如 ∞\\infty |
    | 闭区间 | \[a, b\] | `[a, b]` | \[a,b\]\[a, b\] | 闭区间，例如 \[a,b\]\[a, b\] |
    | 开区间 | (a, b) | `(a, b)` | (a,b)(a, b) | 开区间，例如 (a,b)(a, b) |
    | 矢量 | → | `\vec` | ⃗\\vec{} | 矢量，例如 a⃗\\vec{a} |
    | 微分 | d | `d` | dd | 微分，例如 dydx\\frac{dy}{dx} |
    | 数列求和 | ⋯ | `\cdots` | ⋯\\cdots | 序列，例如 1,2,⋯ ,n1, 2, \\cdots, n |
    | 积分 | ∫ | `\int` | ∫\\int | 积分，例如 ∫abf(x) dx\\int\_a^b f(x) \\, dx |
    | 重积分 | ∬ | `\iint` | ∬\\iint | 重积分，例如 ∬Df(x,y) dA\\iint\_D f(x, y) \\, dA |
    | 复积分 | ∮ | `\oint` | ∮\\oint | 复积分，例如 ∮Cf(z) dz\\oint\_C f(z) \\, dz |
    | 向量场散度 | ∇· | `\nabla \cdot` | ∇⋅\\nabla \\cdot | 散度，例如 ∇⋅F⃗\\nabla \\cdot \\vec{F} |
    | 向量场旋度 | ∇× | `\nabla \times` | ∇×\\nabla \\times | 旋度，例如 ∇×F⃗\\nabla \\times \\vec{F} |
    | 矩阵 | \[\] | `[]` | \[\]\[\] | 矩阵，例如 $A = |

+   **注音与标注**

    | 符号名称 | 符号 | LaTeX 语法 | 渲染效果 | 用途示例 |
        | --- | --- | --- | --- | --- |
    | 数学框 |  | `\boxed{}` | \\boxed{} | 突出显示，例如 a+b\\boxed{a + b} |
    | 上标 |  | `^{}` | ^{ } | 上标，例如 a2a^{2} |
    | 下标 |  | `_` | \_{} | 下标，例如 aia\_{i} |
    | 带括号的上标和下标 |  | `_{}` 和 `^{}` | \_{}^{ } | 带括号的上标和下标，例如 aija\_i^j |
    | 行间公式 |  | `\[ \]` | \\\[ \\\] | 行间公式，例如 \\\[ \\sum\_{i=1}^n i \\\] |
    | 内联公式 |  | `$ ... $` | `$ ... $` | 内联公式，例如 E\=mc2E = mc^2 |
    | 分数 |  | `\frac{}{}` | \\frac{}{} | 分数，例如 ab\\frac{a}{b} |
    | 开平方根 |  | `\sqrt{}` | $\\sqrt{}\` | 开平方根，例如 4\\sqrt{4} |
    | 开 n 次方根 |  | `\sqrt[n]{}` | $\\sqrt\[n\]{}\` | 开 n 次方根，例如 83\\sqrt\[3\]{8} |
    | 向量 | → | `\vec{}` | $\\vec{}\` | 向量，例如 v⃗\\vec{v} |
    | 重音 | ̅ | `\overline{}` | $\\overline{}\` | 重音，例如 AB‾\\overline{AB} |
    | 波浪符 | ˜ | `\tilde{}` | $\\tilde{}\` | 波浪符，例如 a~\\tilde{a} |
    | 波浪符（向量） |  | `\widetilde{}` | $\\widetilde{}\` | 波浪符（向量），例如 AB~\\widetilde{AB} |
    | 括号 |  | `\left( \right)` | ()\\left( \\right) | 括号，例如 (x+y)\\left( x + y \\right) |
    | 花括号 |  | `\left\{ \right\}` | {}\\left\\{ \\right\\} | 花括号，例如 {x∣x\>0}\\left\\{ x \\mid x > 0 \\right\\} |
    | 方括号 |  | `\left[ \right]` | \[\]\\left\[ \\right\] | 方括号，例如 \[x+y\]\\left\[ x + y \\right\] |
    | 大于符号下的注释 |  | `\underset{}{\geq}` | ≥\\underset{}{\\geq} | 注释在符号下，例如 ≥a\\underset{a}{\\geq} |
    | 小于符号上的注释 |  | `\overset{}{\leq}` | ≤\\overset{}{\\leq} | 注释在符号上，例如 ≤b\\overset{b}{\\leq} |
    | 积分上面的注释 |  | `\overset{}{\int}` | ∫\\overset{}{\\int} | 注释在积分符号上，例如 ∫b\\overset{b}{\\int} |
    | 对数上的注释 |  | `\log_{}` | $\\log\_{}\` | 注释在对数符号上，例如 log⁡ax\\log\_{a} x |
    | 三角函数注音 |  | `\sin^{-1}{}` | $\\sin^{-1}{}\` | 反三角函数，例如 sin⁡−1x\\sin^{-1}{x} |
    | 单位向量 | `\hat{}` | $\\hat{}\` | 单位向量，例如 u^\\hat{u} |
    | 平方根的上下标 |  | `\sqrt[x]{y}` | yx\\sqrt\[x\]{y} | 平方根上下标，例如 an\\sqrt\[n\]{a} |
    | 重音和波浪符 | ̅ ˜ | `\bar{}` 和 `\tilde{}` | ˉ‘\\bar{}\` \\tilde{}\` | 重音和波浪符，例如 xˉ\\bar{x} y~\\tilde{y} |
    | 上标注释 |  | `x^2` 和 `y^3` | x2x^2 y3y^3 | 上标注释，例如 x2x^2 y3y^3 |
    | 角度符号 | ∠ | `\angle{}` | $\\angle{}\` | 角度，例如 ∠ABC\\angle{ABC} |


##### 用法示例

1.  数学框
    +   a+b\\boxed{a + b} 突出显示 a+ba + b。
2.  上标和下标
    +   a2a^{2} 表示 aa 的平方。
    +   aia\_{i} 表示 aa 的第 ii 个元素。
    +   aija\_i^j 表示 aa 的第 ii 个元素的 jj 次方。
3.  行间和内联公式
    +   行间公式：∑i\=1ni\\sum\_{i=1}^n i 。
    +   内联公式：E\=mc2E = mc^2。
4.  分数和开方
    +   ab\\frac{a}{b} 表示分数 aa 除以 bb。
    +   4\\sqrt{4} 表示 44 的平方根。
    +   83\\sqrt\[3\]{8} 表示 88 的三次方根。
5.  向量和重音
    +   v⃗\\vec{v} 表示向量 vv。
    +   AB‾\\overline{AB} 表示线段 ABAB 的重音。
6.  括号和花括号
    +   (x+y)\\left( x + y \\right) 表示括号内的 x+yx + y。
    +   {x∣x\>0}\\left\\{ x \\mid x > 0 \\right\\} 表示满足 x\>0x > 0 的集合。
7.  符号上下的注释
    +   ≥a\\underset{a}{\\geq} 表示大于等于符号下的注释 aa。
    +   ≤b\\overset{b}{\\leq} 表示小于等于符号上的注释 bb。
8.  积分和对数上的注释
    +   ∫b\\overset{b}{\\int} 表示积分符号上的注释 bb。
    +   log⁡ax\\log\_{a} x 表示底数为 aa 的对数。
9.  单位向量
    +   u^\\hat{u} 表示单位向量 uu。
10.  三角函数注音
     +   sin⁡−1x\\sin^{-1}{x} 表示 xx 的反正弦函数。
11.  角度符号
     +   ∠ABC\\angle{ABC} 表示角 ABCABC。

+   **省略号、空白间隔、分界符表格**

    | 符号名称 | 符号 | LaTeX 语法 | 渲染效果 | 用途示例 |  |  |  |  |  |  |
        | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
    | 省略号 | ⋯ | `\cdots` | ⋯\\cdots | 水平省略号，例如 1,2,⋯ ,n1, 2, \\cdots, n |  |  |  |  |  |  |
    | 省略号（垂直） | ⋮ | `\vdots` | ⋮\\vdots | 垂直省略号，例如 123⋮⋮⋮nn+1n+2\\begin{matrix} 1 & 2 & 3 \\\\ \\vdots & \\vdots & \\vdots \\\\ n & n+1 & n+2 \\end{matrix} |  |  |  |  |  |  |
    | 省略号（对角线） | ⋱ | `\ddots` | ⋱\\ddots | 对角线省略号，例如 a11a12⋯a1na21a22⋯a2n⋮⋮⋱⋮am1am2⋯amn\\begin{matrix} a\_{11} & a\_{12} & \\cdots & a\_{1n} \\\\ a\_{21} & a\_{22} & \\cdots & a\_{2n} \\\\ \\vdots & \\vdots & \\ddots & \\vdots \\\\ a\_{m1} & a\_{m2} & \\cdots & a\_{mn} \\end{matrix} |  |  |  |  |  |  |
    | 四分之一间隔 |  | `\,` |  \\, | 四分之一间隔，例如 a ba\\,b |  |  |  |  |  |  |
    | 半间隔 |  | `\:` |  \\: | 半间隔，例如 a ba\\:b |  |  |  |  |  |  |
    | 全间隔 |  | `\;` |   \\; | 全间隔，例如 a  ba\\;b |  |  |  |  |  |  |
    | 两倍全间隔 |  | `\quad` | \\quad | 两倍全间隔，例如 aba\\quad b |  |  |  |  |  |  |
    | 四倍全间隔 |  | `\qquad` | \\qquad | 四倍全间隔，例如 aba\\qquad b |  |  |  |  |  |  |
    | 非断行空白 |  | `\!` |  ⁣\\! | 非断行紧密空白，例如 a ⁣ba\\!b |  |  |  |  |  |  |
    | 换行 |  | `\\` | `\\` | 换行，例如 aba \\\\ b |  |  |  |  |  |  |
    | 分隔符（矩形括号） |  | `\left[ \right]` | \[\]\\left\[ \\right\] | 矩形括号，例如 \[a+b\]\\left\[ a + b \\right\] |  |  |  |  |  |  |
    | 分隔符（圆括号） |  | `\left( \right)` | ()\\left( \\right) | 圆括号，例如 (a+b)\\left( a + b \\right) |  |  |  |  |  |  |
    | 分隔符（花括号） |  | `\left\{ \right\}` | {}\\left\\{ \\right\\} | 花括号，例如 {a+b}\\left\\{ a + b \\right\\} |  |  |  |  |  |  |
    | 分隔符（竖线） |  | \`\\left | \\right | \` | $\\left | \\right | $ | 竖线，例如 $\\left | a + b \\right | $ |
    | 分隔符（双竖线） |  | `\left| \right|` | ∥∥\\left\\| \\right\\| | 双竖线，例如 ∥a+b∥\\left\\| a + b \\right\\| |  |  |  |  |  |  |
    | 分隔符（上下括号） |  | `\left\langle \right\rangle` | ⟨⟩\\left\\langle \\right\\rangle | 上下尖括号，例如 ⟨a+b⟩\\left\\langle a + b \\right\\rangle |  |  |  |  |  |  |
    | 绝对值 |  | \`\\left | \\right | \` | $\\left | \\right | $ | 绝对值，例如 $\\left | a \\right | $ |
    | 积分上下限分隔符 |  | \`\\left. \\right | \` | $\\left. \\right | $ | 积分上下限分隔符，例如 $\\int\_{a}^{b} \\left. f(x) \\right | \_{a}^{b}$ |  |  |  |


##### 用法示例

1.  省略号

    +   水平省略号：1,2,⋯ ,n1, 2, \\cdots, n 表示序列从 1 到 n 的省略。
    +   垂直省略号：123⋮⋮⋮nn+1n+2\\begin{matrix} 1 & 2 & 3 \\\\ \\vdots & \\vdots & \\vdots \\\\ n & n+1 & n+2 \\end{matrix} 表示矩阵中的垂直省略。
    +   对角线省略号：a11a12⋯a1na21a22⋯a2n⋮⋮⋱⋮am1am2⋯amn\\begin{matrix} a\_{11} & a\_{12} & \\cdots & a\_{1n} \\\\ a\_{21} & a\_{22} & \\cdots & a\_{2n} \\\\ \\vdots & \\vdots & \\ddots & \\vdots \\\\ a\_{m1} & a\_{m2} & \\cdots & a\_{mn} \\end{matrix} 表示矩阵中的对角线省略。
2.  空白间隔

    +   四分之一间隔：a ba\\,b 表示 a 和 b 之间的四分之一间隔。
    +   半间隔：a ba\\:b 表示 a 和 b 之间的半间隔。
    +   全间隔：a  ba\\;b 表示 a 和 b 之间的全间隔。
    +   两倍全间隔：aba\\quad b 表示 a 和 b 之间的两倍全间隔。
    +   四倍全间隔：aba\\qquad b 表示 a 和 b 之间的四倍全间隔。
    +   非断行紧密空白：a ⁣ba\\!b 表示 a 和 b 之间的紧密空白，且不换行。
3.  分隔符

    +   矩形括号：\[a+b\]\\left\[ a + b \\right\] 表示包含 a + b 的矩形括号。

    +   圆括号：(a+b)\\left( a + b \\right) 表示包含 a + b 的圆括号。

    +   花括号：{a+b}\\left\\{ a + b \\right\\} 表示包含 a + b 的花括号。

    +   竖线：∣a+b∣\\left| a + b \\right| 表示包含 a + b 的竖线，用于绝对值或范数。

    +   双竖线：∥a+b∥\\left\\| a + b \\right\\| 表示包含 a + b 的双竖线，用于范数。

    +   上下尖括号：⟨a+b⟩\\left\\langle a + b \\right\\rangle 表示包含 a + b 的上下尖括号。

    +   绝对值：∣a∣\\left| a \\right| 表示 a 的绝对值。

    +   积分上下限分隔符：∫abf(x)∣ab\\int\_{a}^{b} \\left. f(x) \\right|\_{a}^{b} 表示积分上下限的分隔符。


+   大型数学运算符

    | 数学运算符 | Markdown 语法 | 渲染结果 |
        | --- | --- | --- |
    | 求和 | `\sum` | ∑\\sum |
    | 积分 | `\int` | ∫\\int |
    | 积分（四重） | `\iiiint` | \\iiiint\\iiiint |
    | 累计积分 | `\oint` | ∮\\oint |
    | 闭合曲线积分 | `\oint\oint` | ∮∮\\oint\\oint |
    | 平均值 | `\bar{x}` | xˉ\\bar{x} |
    | 低尺度积分 | `\smallint` | ∫\\smallint |
    | 极限 | `\lim` | lim⁡\\lim |
    | 极限（自下而上） | `\lim\limits_{x \to a}` | lim⁡x→a\\lim\\limits\_{x \\to a} |
    | 极限（函数） | `\lim_{x \to a} f(x)` | lim⁡x→af(x)\\lim\_{x \\to a} f(x) |
    | 极限（自上而下） | `\lim\nolimits_{x \to a}` | lim⁡x→a\\lim\\nolimits\_{x \\to a} |
    | 极限（自左向右） | `\lim_{x \to \infty}` | lim⁡x→∞\\lim\_{x \\to \\infty} |
    | 极限（自右向左） | `\lim_{x \to -\infty}` | lim⁡x→−∞\\lim\_{x \\to -\\infty} |
    | 乘积 | `\prod` | ∏\\prod |


#### 常用的数学表达式

以下是一些常用的数学符号和表达式，以及它们的 LaTeX 表达方式：

+   基本运算符

    ```css
    加法：$a + b$
    减法：$a - b$
    乘法：$a \times b$
    除法：$\frac{a}{b}$
    ```

    渲染后的效果：

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/328668b7752e4b0293f9c1aa3b83ffb9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=340&h=250&s=20300&e=png&b=1e1e1e)
+   指数和下标

    ```ruby
    指数：$a^n$
    下标：$a_n$
    ```

    渲染后的效果：

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b8603983f7f4026ba6daa3891518a64~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=262&h=130&s=10297&e=png&b=1e1e1e)
+   分数和根式

    ```ruby
    分数：$\frac{a}{b}$
    平方根：$\sqrt{a}$
    n次方根：$\sqrt[n]{a}$
    ```

    渲染后的效果：

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4e81060fd144ec39f8d6bab7847fbe4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=444&h=194&s=17674&e=png&b=1e1e1e)
+   微积分

    ```ruby
    导数：$\frac{dy}{dx}$
    偏导数：$\frac{\partial y}{\partial x}$
    积分：$\int_a^b f(x) \, dx$
    多重积分：$\iint \limits_{D} f(x,y) \, dx \, dy$
    ```

    渲染后的效果：

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21ec8bae58754d32a9dccfd0fc5e7725~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=552&h=288&s=35161&e=png&b=1e1e1e)
+   求和和极限

    ```ruby
    求和：$\sum_{i=1}^{n} i$
    极限：$\lim_{x \to \infty} f(x)$
    ```

    渲染后的效果：

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/790911c02b3145d6a4d158b174805c55~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=456&h=138&s=16345&e=png&b=1e1e1e)
+   矩阵

    ```ruby
    矩阵：
    $$
    \begin{bmatrix}
    a & b \\
    c & d
    \end{bmatrix}
    $$
    ```

    渲染后的效果：

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5781324e5a864f2ea48ed44bfd149e51~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=402&h=214&s=10276&e=png&b=1e1e1e)
+   操作符

    ```ruby
    希腊字母：$\alpha, \beta, \gamma, \delta$
    集合：$\in, \notin, \subset, \supset$
    符号：$\bigcup, \bigcap, \int, \sum$
    逻辑符号：$\land, \lor, \neg, \implies$
    ```

    渲染后的效果：

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9803097540b14363a4021e8787c25dc6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=572&h=238&s=29489&e=png&b=1e1e1e)

#### 综合示例

+   示例一： 假设有一个函数 f(x)\=x2f(x) = x^2，我们希望计算从 aa 到 bb 的积分： ∫abx2 dx\=\[x33\]ab\=b33−a33 \\int\_{a}^{b} x^2 \\, dx = \\left\[ \\frac{x^3}{3} \\right\]\_{a}^{b} = \\frac{b^3}{3} - \\frac{a^3}{3}

    ```ruby
    假设有一个 2x2 矩阵 $A$：
    
    $$
    A = \begin{pmatrix}
    a & b \\
    c & d
    \end{pmatrix}
    $$
    ```

    渲染后的效果： ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16a1de3a30464335afdbe229a2376085~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1154&h=242&s=40405&e=png&b=1e1e1e)

+   示例二

    ```ruby
    1. 设正整数m，满足$2m+3n=mn+2$ ，则 $mn$ 的最大值是?
    
    解: 
    
        $\because 2m+3n=mn+2$​
    
        $\therefore mn+2-2m+3n=0$​
    
        得 $mn-2m-3n=-2$
    
        给这个方程增加一个常数项，以便因式分解。注意到我们可以通过添加和减去6进行因式分解：
    
        $mn-2m-3n+6=4$
    
        因此可以重写为:
    
        $(m-3)(n-2)=4$​
    
        4的所有正因数$(a, b) \Longrightarrow (1,4),(2,2)(4,1)$
    
        把这些因数代入方程$(m-3,n-2)$ 中，得到：
    $$
    \begin{cases}
    m-3=1 & \text{m=4} \\
    n-2=4 & \text{n=6} \
    \end{cases}
    \
    \text{或者}
    \begin{cases}
    m-3=2 & \text{m=5} \\
    n-2=2 & \text{n=4} \
    \end{cases}
    \
    \text{或者}
    \begin{cases}
    m-3=4 & \text{m=7} \\
    n-2=1 & \text{n=3} \
    \end{cases}
    $$
    对应每组 $(m,n)$，我们计算 $mn$：
    $$
    \begin{align*} 
    mn = 4 \times 6 &= 24 \\ 
    mn = 5 \times 4 &= 20 \\ 
    mn = 7 \times 3 &= 21 
    \end{align*}
    $$
    所以最大值为 $24$
    ```

    渲染后的效果如下：

    ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4ec4248fd7d40828196a7a3c635f97a~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1444&h=1302&s=215355&e=png&b=1e1e1e)

## Markdown 扩展和插件

根据不同的平台和工具，Markdown 还可以通过插件和扩展提供更多功能。例如：

+   [Mermaid](https://mermaid.js.org/intro/getting-started.html "https://mermaid.js.org/intro/getting-started.html")：用于绘制流程图和图表。
+   [MathJax](https://www.mathjax.org/ "https://www.mathjax.org/")：高级数学公式支持。
+   [Diagram](https://support.typora.io/Draw-Diagrams-With-Markdown/ "https://support.typora.io/Draw-Diagrams-With-Markdown/")：绘制各类图表（如序列图、甘特图）。

工具：

+   [typora](https://typora.io/ "https://typora.io/") 一款 Markdown 编辑器和阅读器。

+   [marktext](https://www.marktext.cc/ "https://www.marktext.cc/") 简洁优雅的 Markdown 编辑器，注重速度和可用性。

+   [markdown-online-editor](https://github.com/nicejade/markdown-online-editor "https://github.com/nicejade/markdown-online-editor") 支持绘制流程图、甘特图、时序图、任务列表、echarts 图表、五线谱，以及 PPT 预览、视频音频解析、HTML 自动转换为 Markdown 等功能。

+   [Vditor](https://github.com/Vanessa219/vditor "https://github.com/Vanessa219/vditor") 是一款浏览器端的 Markdown 编辑器，支持所见即所得、即时渲染（类似 Typora）和分屏预览模式。
