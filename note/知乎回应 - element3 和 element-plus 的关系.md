看到这个题目，正好可以作为 element3 核心开发者来回答一下最近做的一些事

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/717fa96b9bd645dc891d98a53e370b55~tplv-k3u1fbpfcp-watermark.image)

其实做 element3 我们的初衷只有两个：
- 为社区做点技术输出
- 为开发者提供更优质的教学资源

最早 @蜗牛老湿-大圣 发起这个项目的时候，他完成了最初的适配工作，我只是参与了几个组件开发，后来他找我聊了一下

聊了一下他对 element3 的定位和想法，我也觉得为了教育的 UI 框架很有意义，并且我之前的工作经历就是做组件库，所以一路发展到了现在

## element3 做了什么
先来回顾一下从七月份到现在我们都做了什么
### 重构逻辑

最初我们的想法是先让项目跑起来，所以直接在原有代码的基础之上进行升级，适配 vue3。

当时采用的技术方案：
- 先添加测试安全网
- 在重构其组件内部逻辑

最初 element-ui 是使用自己编写的逻辑来测试组件，我们直接用 [VTU](https://github.com/vuejs/vue-test-utils-next) 来替换

基本的做法是用 vtu 来重写之前的组件逻辑。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eca8ddf4195a436980e5154a0c7f4e40~tplv-k3u1fbpfcp-watermark.image)

在有了测试安全网的保障下，我们使用 composition api 对原有组件逻辑进行重构

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/329d0ad903884cdb8bf63dc32b3d084b~tplv-k3u1fbpfcp-watermark.image)

这样我们保障了组件可以快速的跑起来



### 文档升级

我们对官网文档采用的方案是更新所有 demo，把之前的 options api 全部升级到 composition api 写法

方便让新用户可以更直观的看到 composition api 的用法

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b686778449794ea99a687f71c1c48384~tplv-k3u1fbpfcp-watermark.image)

并且申请了新的域名方便用户访问： https://element3-ui.com/

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1b8a013bdb84b2f8ee77faf8f05f729~tplv-k3u1fbpfcp-watermark.image)


### 使用 rollup 打包

在 build 上，我们放弃了 webpack，选择了 rollup 

并且支持了多个环境：
- cjs（nodejs坏境）
- esm-browser(浏览器环境)
- esm-bundler(给打包工具准备的)
- gloabl (全局引入)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c62b8bc2c0845c596117a7aca633a04~tplv-k3u1fbpfcp-watermark.image)

### 新的 logo

我们还设计了 element3 的 logo 

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35665677218e4f48a1b808f9bc9e258a~tplv-k3u1fbpfcp-watermark.image)

并且搞了点小周边

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e60ed2d2237d44b281c3718608984ec0~tplv-k3u1fbpfcp-watermark.image)

## 现在的成绩

### 组件
我们完成了百分之九十五的组件逻辑的重构，用户直接就可以使用，并且支持多个环境，以及按需加载等使用方式

暂时不支持的组件：
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33e000157c874746b2f4796b24258d9b~tplv-k3u1fbpfcp-watermark.image)
> 欢迎 pr

#### 特别说明
table 组件的 pr 是借鉴的 element-plus 的实现

因此我们特意在 README.md 上标出

后面我们会重写掉

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e63a6ce8b7e14e0c8df2237581ad2655~tplv-k3u1fbpfcp-watermark.image)

### 文档
所有组件文档的使用 demo 全部使用 composition api 重写完成

> 如果还有遗漏，欢迎大家指出

### 教程产出数量
除了在做好开发的基础上，还产出了大量的教程


- [【element3-开发日记】手摸手教你重写 Button 组件](https://juejin.cn/post/6898959238576472077)
- [vue3 避坑指南：解决 dev 和 prod 坏境下逻辑不一致问题](https://juejin.cn/post/6899432348266283022)
- [使用 TDD 开发组件 --- Notification (上)](https://juejin.cn/post/6844904013503152141)
- [使用 TDD 开发组件 --- Notification （下）](https://juejin.cn/post/6844903971258105863)
- [拿下vue3你要做好这些准备](https://juejin.cn/post/6866373381424414734)
- [Element3开发内幕 - Vue CLI插件开发](https://juejin.cn/post/6899334776860180494)
- [Element3.0升级日记 - TimeLine组件](https://juejin.cn/post/6867865667089989639)
- [跟我一起编写Vue3版ElementUI](https://juejin.cn/post/6864462363039531022)

我们还有团队语雀，里面也有大量的教程
[花果山](https://www.yuque.com/hugsun)

### issues 和 pr 
现有的 issues 和 pr 的处理，数量堆积不超过 10 个

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fda265c99d1d47d197b5ca98f6e0fe8b~tplv-k3u1fbpfcp-watermark.image)


服务好每个开发者的问题以及需求也是我们一直坚持的理念


### 1.9k star
其实 element3 的曝光率并不高，只在掘金和知乎上有所谈起

但是我们目前依旧收获了 1.9k star ，算是赢得了大家的一些认可

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/594585b32bb349aba2163aa12abd447d~tplv-k3u1fbpfcp-watermark.image)

### 社区
我们有了小 1500+ 的开发群

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56e8c66a5bdd4b69a5bb9d1c2f6563aa~tplv-k3u1fbpfcp-watermark.image)
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b9b62f7be7d4751a0c442fbc4fb7488~tplv-k3u1fbpfcp-watermark.image)

并且吸纳 10+ 的社区活跃贡献者。
## 未来规划

### 组件逻辑
组件逻辑会使用 TDD 开发的方式进行重写

首先之前从 options api 重构成 composition api 实现的不够优雅，大多数是从 options api 硬改过来的，没有从整体考虑。

其次有了 composition api 后，可以继续抽离公共逻辑，沉淀出一套 composition api 的库

最后我对原有组件测试不满意，覆盖率需要达到至少 80% 以上。

这样不只是用户使用起来更安心，在后续的扩展上也会更加放心。

> 可参考这篇文章学习 TDD [手摸手教你重写 Button 组件](https://juejin.cn/post/6898959238576472077)


### 教程
教程是专属于我们教研团队的特点，除了正常的开发节奏，每一个组件都会编写教程来帮助广大前端开发者学习

并且除了文字，还会出配套视频。

目的很简单，就是帮助大家好好学习，天天向上


### 官网
现有的官网 demo 虽然已经比较明确了，新手一看就可以懂，但是还不够

后续我们会采用 stroybook 重写组件 demo，让新手用户的体验更加友好

[stroybook](https://storybook.js.org/)

### monorepo
现在的主流库都上了 monorepo 来管理项目

目前的技术方案是： Lerna 配合 Yarn

后面我们会用 npm v7 的 [workspace](https://docs.npmjs.com/cli/v7/using-npm/workspaces) 来做升级

### TS
引入 typescript 来重构组件

### 面向未来的探索

对未来的探索，因为有了 composition api ，我们可以大胆想象一下未来的组件库发展方向

比如是否可以让用户重写组件内部逻辑

就像 css 一样，定义一个 class 就可以覆盖重写样式

class 其实就是接口，样式就是逻辑。

### 开放

除了正常的研发和用户使用之外，我们还会积极做到更加开放

比如：
- 组织线下活动
- 提供技术支持等

## 总结

以上就是我们 element3 做的事以及未来要做的事。

我们只想踏踏实实做事，为社区贡献自己的一份力量

并且这件事也会帮助我们自己成长，所以会一直坚持下去

如果你有想法，也可以一起加入到我们 
 





