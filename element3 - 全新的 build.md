## 背景
element3 的那套 build 已经过时

需要重新设计一套全新的 build 流程

比如说支持 treeshaking

## tasking
- 是用 webpack 还是用 rollup？
	- 目前流行的组件库是用的 webpack 还是 rollup？
	- 可以参考一下

## 目标
- 只需要构建出一个 button 组件，用户可以用即可
- 在这个基础上支持按需导出


## how
直接创建一个 button 组件

然后写了一个最简单的 rollup.config.js 看看能不能把组件打包出去

着时候需要用到插件 rollup-plugin-vue

来处理 .vue 的文件

着时候遇到了一个坑，就是不能再 .vue 里面写 style ，如果写了样式的话，那么直接就报错了

查了查文档发现，还需要安装一个插件来专门处理 css 样式 rollup-plugin-scss 

安装完解决了这个问题，但是这个需要额外安装一个插件去处理 css 样式这个行为很让人不解，理论上应该由 rollup-plugin-scss 都处理完才对

现在可以正常的打包出去啦。 

目前的 rollup.config.js 

```
import packageJson from "./package.json";
import vuePlugin from "rollup-plugin-vue";
import scss from "rollup-plugin-scss";
export default {
  input: "src/index.js",
  output: [
    {
      file: packageJson.main,
      format: "cjs",
      sourcemap: true,
    },
    {
      file: packageJson.module,
      format: "esm",
      sourcemap: true,
    },
  ],
  plugins: [
    vuePlugin({
      css: true,
    }),
    scss(),
  ],
};

```

==但是现在打包出去还是有问题，比如 css 样式单独打包到了一个文件，这样用户在用的时候需要单独去引入这个 css 文件，有没有更好的解决方案呢？==

还有一个问题，如果是用 vite 坏境下导入打包好的组件的话，通过 import {Button} from "vue-component-library" 就不行，找不到 Button ，但是明明导出的就是一个对象，但是就是解构不了

然后我用 vue-cli 试了试，发现是没有问题的，那问题应该是在 vite 那边



暂时先不管那么多，先按照教程继续走

其中用到了以下这些插件：

- rollup-plugin-peer-deps-external
	- 用来自动把 peerDependencies 里面的依赖添加到 rollup.config.js 里面的  External 选项内
	- 放到 External 内的库是不会打包到 bundle.js 内的
- @rollup/plugin-node-resolve @rollup/plugin-commonjs
	- 我们在项目里面需要引入第三方库，而着两个插件就是为了解决这个问题的
	- 一个是解析引入 node_modules 的第三方库
	- 一个是支持 commonjs 的调用方式 ，因为默认 rollup 只支持 esm
	- 加了着两个库之后，就会把用到的这个第三方库打包到自己的 bundle.js 内了，会增加大小，
	- 但是如果不用着两个库的话，就需要用户安装所有的依赖，不能直接拿到 bundle.js 就用了 ，着就是区别
- @rollup/plugin-json 
	- 用来解析 json 文件


## 集成 storybook
遇到问题了，storybook 暂时还不支持 vue3 ，但是 element-plugs 是怎么做到的呢？

去看看他们的代码 

他们的代码nb 啊，自己写了一个自定义的处理器，来专门处理 vue3 

他们的代码值得学习的地方有挺多，可以先摸清他们的框架套路，然后用到自己的库里面

着就是学习的过程 哈哈哈哈

==集成 storybook 这件事可以先等一等==

毕竟还是官方的版本最靠谱一些




## 调研发布的方式
现在已知有两种发布方式

一种是使用 monorepo 的形式，一个组件就是一个对应的包

第二种是组件库是一个包，可以指定对应的路径来引入对应的组件

这两个的优缺点是什么呢？

需要看看各大著名的组件库都采用的哪种方式



 突然明白为什么有的库要带上整个所有的代码了， 比如 整个 lib 文件夹,或者 es 文件夹
 
 比如 ant-design-vue ，在 node_modules 里面带了 lib 文件夹和 es 文件夹 还有对应的打包到一个文件内的形式
 
 着是因为用户可以单独引入，，为了 commonjs 用户准备的
 
 原来如此！ 




## 调研社区内的组件库
策略就是只参考使用 rollup 打包的组件库


### 调研 - vuetify
- vuetify 采用 vue-cli 自动帮助用户安装，这个思路还是很好的，什么乱七八糟的依赖统统交给命令，一个命令搞定
   - 后面可以做个 vue-cli 安装功能，一键安装组件库



### 调研 - bootstrap-vue
- 这个库看起来打包是用的 rollup 
- 看起来比较干净，可以借鉴一下


## 需要参考的库

- bootstrap-vue 
	- [文档](https://bootstrap-vue.org/docs)
	
	
	
## 组件库的使用方式
- import 导入
- script 导入
- 调用时机
	When webpack 2+, Rollup, or other modern build tools are used, they will pick up on the module build. Legacy applications would use the main build, and the unpkg build can be used directly in browsers. In fact, the unpkg cdn automatically uses this when someone enters the URL for your module into their service!


## 组件库打包要提供的几个点
- commonjs 
	- 不走 esm 的用户就走 commonjs
	- 走 package.json/ main
- esm
	- webpack2 和 rollup 用户会使用这种版本 
	- 走 package.json / module
- iife(umd)
	- unpkg 
	- iife
	- 全局使用 通过 script引入

- 入口文件也是必须的
	- 用来统一注册组件
		- 可以参考 sfc-init-components 项目


## 思考
可以采用 TDD 的思想，先考虑用户的使用方式是什么样子的，也就是先确认接口

> 没错，打包后的结果也是其使用接口。

确定好这件事之后，在去考虑如何去实现

这里使用拆分的思想来实现所有功能（tasking）

- [x] 搞定 esm 形式的打包
	- [ ] 先把 element3 里面的 button 逻辑拿过来跑一下试试
- [ ] 搞定 commonjs 形式的打包


## 问题
打包出来后的文件，如果是 npm i ../xxxx 的这种方式引入的话，就报错，vue-next 执行顺序乱了，但是如果把打包之后的文件直接放到 test 内执行的话，就没有问题，说明什么，说明打包后的文件格式是没有问题的，但是为什么引入的时候就错乱呢？ 

- 可以尝试放到 npm 上引入看看如何
- 还有就是坏境和 vue-next 的版本有关系
- rollup-config-vue 这个插件会不会也有问题

但是有问题的话不应该test直接引入打包后的文件就没有问题呀。。。费解了

解决了，发布到 npm 上，然后在引入的话就没有问题了

我擦，npm 引用本地库有问题！


## 集成
流程跑通了，剩下的就是把这套流程集成进 element3 项目里

其实还有一个关于样式的问题没处理，css 和 scss 要怎么处理呢?

先别管别的了，直接在 element3 上开一个分支，然后跑一下试试看，

遇到啥问题解决啥问题就完了。


### .vue 后缀没有办法解析的问题

rollup 必须要知道这个文件是 .vue 后缀的，不然就解析不了

着怎么搞？

它没有办法确定这个导入的文件是不是 .vue 文件

先看看社区里面有没有解决方案

不行的话，研究研究如何给 rollup 写插件来解决这个问题


### 如何 build scss 文件呢？

暂时还不知道。。。

参考一下 bootstrap-vue 库

bootstrap-vue 这个库是通过 node-sass + post-css 来单独编译的 scss ，和 js 一样也需要一个入口文件

不过看起来可以复用之前 element 打包 css 的逻辑

我擦擦，这个 css 是用 gulp 构建的，，太古老了

现在可以通过直接在入口 index.js 直接引入 css ，打包的时候自动就会打包进去了

但是我想参考 vue3 的做法，css 应该作为一个单独的库存在，然后引入的时候也只是通过 npm 导入包的形式来引入，我去看看 vue3

尝试了一下，不能只导入 css，还需要 css 依赖的资源，比如这里依赖的是 font 文件

所以还是用之前的逻辑把

之前的逻辑是打包完 css 之后，把它复制到 lib 文件夹 下，然后发布的时候把 lib 也发布上去即可


## 总结
- [[库打包需要支持的坏境]]


## 参考资料
- [深入学习rollup来进行打包](https://www.cnblogs.com/tugenhua0707/p/8179686.html)
- [Creating a React Component Library using Rollup, Typescript, Sass and Storybook](https://blog.harveydelaney.com/creating-your-own-react-component-library/)
- [深入学习rollup来进行打包](https://www.cnblogs.com/tugenhua0707/p/8179686.html#_labe3)
- [packaging-sfc-for-npm](https://vuejs.org/v2/cookbook/packaging-sfc-for-npm.html)