## what

哈喽，大家好，今天给大家推荐一个学习 vue3 源码的最佳利器

也是阮一峰老师在[第 144 期周刊](http://www.ruanyifeng.com/blog/2021/01/weekly-issue-144.html)里面推荐的

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1fa1396875d4b3dbd3a9b51479c5f3b~tplv-k3u1fbpfcp-watermark.image)

当然投稿人是我，我也是这个库的作者

下面我就来介绍一下当初为什么做这个库，并且这个库应该如何食用

[mini-vue](https://github.com/cuixiaorui/mini-vue)

[可观看视频版：使用详细指南](https://www.bilibili.com/video/BV1Zy4y1J73E/)

## why

vue3 作为目前最火的技术，大家除了学会如何使用以外，肯定是想在深入到源码里面，看看这些 nb 的功能到底是如何实现的，或者是增加自己的核心竞争力搞懂原理，面试的时候装个13

但是当我们打开 vue3 的源码之后你会发现，代码量是如此之多。这个源码到底该从何读起。虽然 vue3 代码的可读性是很高的，但是架不住代码量大呀！！！

举个例子，在 renderer.ts 中 [baseCreateRenderer](https://github.com/vuejs/vue-next/blob/master/packages/runtime-core/src/renderer.ts#L424) 函数就有 1800 行之多的代码量

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92be878f352e4a68add2464c234f20cb~tplv-k3u1fbpfcp-watermark.image)

> 真让人头秃

第一次看到这个函数的同学肯定是一脸懵逼，这要怎么看嘛？我的秀发还能不能保住了？可能这个时候就有好多同学已经被劝退了(悄咪咪的关闭了 vscode)
> 说的是不是你 -_-!

对于这么大的代码量，我们可以利用**分而治之**的思想，去拆小，然后逐一击破

那其实如果我们好好分析分析源码的话，你就会发现有好多逻辑其实是处理边缘 case 或者是单独处理 dev 环境的，这些逻辑其实在了解 vue3 核心逻辑的时候是不用关心的

比如说这些画红框的逻辑：
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fcf15c001fdf459f833fbac15b8bd382~tplv-k3u1fbpfcp-watermark.image)

全部都是处理 dev 坏境下的行为，在我们了解核心逻辑的时候这些点是可以先被忽略掉的
> 我可没说这些逻辑不重要喔，只不过可以后面在了解。先聚焦关注点

那能不能有这么一个库，只会涉及到核心逻辑，去除非核心逻辑，让代码更具备可读性呢？可以更好的理解的？ 

那其实在社区里面就会有这种类型的库，只实现库的核心逻辑，可以让同学们更快速的理解库的核心逻辑，比如一个 mini 版本

但是在社区里面我并没有发现这么一个 mini 版本，所以我就索性撸起袖子自己上

### 为了社区
那其实我为了能让同学们都可以基于 mini-vue 快速的了解 vue3 的核心逻辑我在代码里面重点做了两件事

#### 详细的注释
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16f1e904fbb44118a217b134e7560acf~tplv-k3u1fbpfcp-watermark.image)

在每一个具体的代码上都增加了详细的注释，方便大家可以更快速的理解代码的行为

#### 可视化的运行流程
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a38d1e8f7cdf45c8b6dc84277db0882a~tplv-k3u1fbpfcp-watermark.image)

在每个关键的路径节点上我都给出了 console.log ，这样的话，只需要打开调试台就可以看到详细的运行流程，并且当你想要 debug 代码的时候也可以通过 console.log 快速的定位到具体的代码

### 为了自己

那从我自己的角度而言，看完代码后，除了整理成文章，还有一个更好的方式来验证自己是否真的掌握了

就是自己把功能实现一遍，这对于我自己的收获也是巨大的。因为你要想实现这个功能的话，你必须要先理解，然后才可以把代码写出来。

## how
接着我来介绍一下，如何食用这个库吧

### 下载
首先你需要把库下载到本地 
[库的传送门](https://github.com/cuixiaorui/mini-vue)
```
git clone https://github.com/cuixiaorui/mini-vue.git
```

### README
在 README 里面我详细的记录了目前已经实现的功能

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9fc82c268f0045f39092a4b08587219e~tplv-k3u1fbpfcp-watermark.image)

以及总结出来的脑图

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92244e9205cc47b5a4195679782f5b66~tplv-k3u1fbpfcp-watermark.image)

> 大家可以基于脑图把整个核心流程串起来

### 项目结构
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5bbf0176e20041f7ad805888cdc44d12~tplv-k3u1fbpfcp-watermark.image)

- example 是放置了一些 demo
- lib 是 build 之后的文件（demo 实际执行的文件）
- src 是核心逻辑


### Example 的食用
大家一开始的时候可以先从 example 里面的 demo 入手，先看看一个 demo 是如何运行起来的

先基于 console.log 的输出来 debug 代码

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b162f6614ce640f69677783c4b1f6970~tplv-k3u1fbpfcp-watermark.image)

### 阅读代码
在已经理解完核心运行流程之后，就可以深入到具体的代码里面了，比如说：
- 看看 props 是如何做 diff 的
- 看看 children 是如何处理的

因为有详细的注释，并且只保留了核心逻辑，所以在阅读的时候难度相对是简单许多的。

而且 mini-vue 中所有的函数命名都是和源码中保持一致的，你可以复制函数名然后去 vue3 中搜索来对比学习

当你已经掌握并且理解这个 mini-vue 了，你就可以去看 vue3 的源码了，你会发现读起来真的就没那么难了

## 最后

更详细的 mini-vue 使用教程可以观看[视频版本](https://www.bilibili.com/video/BV1Zy4y1J73E/)

如果这个库可以帮助到你的话，那么希望可以帮我[点个 star](https://github.com/cuixiaorui/mini-vue) 作为开源鼓励（你的鼓励就是我的动力）

后面我每天都更新一点，逐步完善 mini-vue ，希望可以帮助到更多热爱前端的同学

也欢迎去 [issues](https://github.com/cuixiaorui/mini-vue/issues) 中提出你的意见和建议来一起完善 mini-vue