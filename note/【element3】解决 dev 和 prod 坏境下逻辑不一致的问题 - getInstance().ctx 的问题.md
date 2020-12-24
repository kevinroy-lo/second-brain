## 背景
最近在开发 element3 的时候，遇到了一个问题，打包后的代码会报错，和在 dev 坏境下的代码行为不一致。

调研后发现这个问题可能小伙伴们都会遇到 ！！ 所以赶紧写了篇文章记录下来，让大家少掉一点头发

## 发现问题

element3 社区群有小伙伴反馈了一个问题：

组件 Tab 在 dev 坏境下逻辑一切正常，但是在 prod 坏境下就报错了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b10d765e1877479ca66a4b519e0bf47b~tplv-k3u1fbpfcp-watermark.image)

潜意识里面发现这个问题没那么简单，因为之前也有小伙伴说在 prod 坏境下某些组件报错

## 定位问题

赶紧要到 demo，先跑了 dev 环境。嗯，没啥问题

然后又跑了 prod 坏境。 嗯，果然出现问题了

ok， 出现问题就好办了，接着查看了一下报错信息

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/114e9503e4234a1fb127b058a9a4fd7e~tplv-k3u1fbpfcp-watermark.image)

发现在 prod 坏境下 ctx 这个对象的数据结构发生改变了

这是在 dev 坏境下：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa8c490772794592bc0af31e1a4225d7~tplv-k3u1fbpfcp-watermark.image)

这是在 prod 坏境下：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d557b2b92404f22823e0d226329daa9~tplv-k3u1fbpfcp-watermark.image)

好，问题我们已经定位好了，其实问题就已经解决了一半了

那我们剩下的其实只需要弄明白两个点

- 我们怎么能绕过这个问题
- vue3 源码里面是如何处理这个 ctx 的

## 解决问题

我们先来看看如何绕过这个问题

先找到问题代码，在 element3/Tabs.vue 这个组件内

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4da5973b6f9e497aa8e1f0d0645f5d28~tplv-k3u1fbpfcp-watermark.image)

这里的逻辑就是通过 provide 给所有子组件提供当前组件的实例对象

但是问题就发生在这里，我们通过上面的案例已经知道，组件实例对象在 dev 和 prod 坏境下 ctx 对象的数据结构是不一致的。

然后看一下在哪里使用的，找到 element3/TabPane.vue 这个组件内

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a8777b8ead34c5e9bbc2e0a2cf93b8a~tplv-k3u1fbpfcp-watermark.image)

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab92d1fc77014ef7ab95217d2e73897d~tplv-k3u1fbpfcp-watermark.image)
> 这个代码写的真的是丑呀，这也正是我要重写所有组件的原因所在！

那我们怎么绕过这个问题呢？ 我们已知组件实例对象的 ctx 是会变的，那么我们能不能在 Tab 组件中只提供 TabPane 中用到的数据呢？我们只要不用到 ctx 是不是就 ok 了

来 我们先来试一试

稍微统计了一下会涉及到以下几个数据
- tabList
- props.modelValue
- state.activeName
- state.activeIndex

那我们回到 Tab 组件中只提供 TabPane 用到的数据吧

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f35243d07f94b4b8b5d3f6fcdfccc77~tplv-k3u1fbpfcp-watermark.image)

回来 TabPane 改下使用方式

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6eb7282239d472cb839f39b8c114e63~tplv-k3u1fbpfcp-watermark.image)


这样我们就绕过了 ctx 的使用

好，问题修完了 我们打个包 验证一下 有没有问题吧

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79ba55cd39b8465bb3808e8a11efe510~tplv-k3u1fbpfcp-watermark.image)

打包成功，接着把生成的文件放到报错的 demo 里面执行一下

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25c4bcc065d64555889682cfb127ca89~tplv-k3u1fbpfcp-watermark.image)

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63c7ab8de43c49a8b0cb202fc839f86d~tplv-k3u1fbpfcp-watermark.image)

先验证 dev 坏境
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7469d0fe103748b6804429323780bfd2~tplv-k3u1fbpfcp-watermark.image)

控制台安安静静 ，不错不错

在看看 prod 坏境

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f66e80bedcd4263807177f127e1fb7e~tplv-k3u1fbpfcp-watermark.image)

欧耶，完美解决

## 深入问题

好 问题虽然解决了，但是还不能停，我们还要深入探索一下 vue3 的源码，搞懂两个问题

- 源码是怎么基于环境处理的 ctx
- 为什么它要这么处理 ctx

### 源码是怎么基于环境处理的 ctx

要想知道这个问题，其实只要找到生成 ctx 的代码逻辑就可以了

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5bff5eb65818408aa012a6fff86c2852~tplv-k3u1fbpfcp-watermark.image)

很简单的逻辑，就是判断是否为 dev 坏境，然后做逻辑处理

### 为什么它要这么处理 ctx

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/509600c9863c4ba1a4769084272d860c~tplv-k3u1fbpfcp-watermark.image)

基于源码里面的注释我们也已经了解到了

**这个 ctx 就是为了便于在开发模式下通过控制台检查**

**而到了 prod 模式下，它就会变成一个空对象了**

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61f76bbbd41b4d6eb739c17fc86aafc7~tplv-k3u1fbpfcp-watermark.image)

而在 vue 中定义 ctx 的注释中了解到，这个是公共实例的代理 target，并且还包含了一些通过 options 注入的属性以及用户自定义的属性

那我们就验证一下用 options api 写的时候它是什么样子的



![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f0d87edcbd4f47d0b56f89acf4268ea0~tplv-k3u1fbpfcp-watermark.image)

dev 环境下的输出结果

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d1244d16c6a4a4f8af94775f62dd7c7~tplv-k3u1fbpfcp-watermark.image)

prod 坏境下的输出结果
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d5d5a35b42e484281b0ada8e82351b6~tplv-k3u1fbpfcp-watermark.image)

果然 vue 把我们在 methods 中声明好的方法挂载到了 ctx 上，所以我们访问起来还是没有问题的

但是没有看到 count，盲猜的话应该是在对应的 proxy handler 内处理的

其实到这里我又产生了一个问题，就是 vue 里面是怎么处理的这个逻辑。

这个问题就留着下回在讲吧~ 怕你们太长不看

so，我们现在就已经得出了一个结论：

**在以后的开发中千万不要在 setup 中调用 getCurrentInstance().ctx 来获取组件内部数据**

## 总结

最后来总结一下，我们从问题出发，发现问题，定位问题，解决问题，最后深入问题。

并且最终得到了结论：**在 setup 中不可以调用 getCurrentInstance().ctx 来获取组件内部数据，因为在 prod 会被干掉**

你学到了吗？

> 如果这篇文章你有所获的话，请帮我点个赞

---
![](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/058f20e8cee84bdb9c0a62b36dc084e5~tplv-k3u1fbpfcp-zoom-1.image)
- 这是我们[花果山团队](https://www.yuque.com/hugsun)的开源项目 [element3](https://github.com/kkbjs/element3)
- 一个支持 vue3 的前端组件库
---
扫下二维码，马上进群讨论

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60db3f5a808b48ab95870e03d51fda49~tplv-k3u1fbpfcp-watermark.image)















element3devprod