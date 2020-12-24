## 背景
使用 vue-cli 创建的项目，eslint 配置不是很懂，所以想深入分析一下 vue 的 eslint 是如何搞的，正好借这个机会深入了解一下 eslint

## 目标
- 编写简单的 eslint 插件
- 可以看懂 vue 提供的 eslint 插件逻辑
- 输出一篇文章

## 怎么做
在阅读一遍官方的资料

先把最基础的知识点搞定

### 配置
eslint-config-standard

extends 属性值可以省略包名的前缀 eslint-config-。

可共享的配置 是一个 npm 包，它==输出一个配置对象==。要确保这个包安装在 ESLint 能请求到的目录下

本质就是导出一个 eslint config 对象，可以和别人共享这些 eslint config，别人就可以通过 extends 来使用。

这里有个扩展是：多个配置在同一个 npm 包中

着时候就需要指定一下到底要用哪个配置了

直接指定对应的文件即可

比如你的配置是写在库里面叫做 my-special-config.js 的文件里

引入的时候直接向下面这样引入即可

```
{
    "extends": "myconfig/my-special-config"
}
```

### 解析器
Processors in Plugins
你也可以创建插件告诉 ESLint 如何处理 JavaScript 之外的文件。


### 插件

plugins 属性值 可以省略包名的前缀 eslint-plugin-。

插件 是一个 npm 包，通常==输出规则==。一些插件也可以输出一个或多个命名的 配置。要确保这个包安装在 ESLint 能请求到的目录下。

- 可以暴露额外的 rules
- 可以暴露额外的 Environments
- 可以创建一个处理器 Processors
- 可以通过 cnofigs 指定打包的配置

看起来是啥都可以看，eslint 给提供了一个插件生成器，用一下看看

看了一下就是给你生成对应的目录结构，那么先可以看看如果创建一个自定义的 rule

研究完这些之后，看看别人都是如何写的 plugin

### 创建规则

本质是利用 AST 来检测错误，然后基于对应的 hook 来处理逻辑

而且还发现了 fix 的逻辑，就是实现 fix 接口，修复相对应的问题

AST 暂时不去深入了解，后面可以深入了解一下 AST ，做一下小工具

---

好，大概都了解的差不多了。可以看看别人是如何写的 plugin 了

这里看的 plugin 是 eslint-plugin-vue

看看 vue 官方的 eslint plugin 是怎么写的

入口文件是 lib/index.js 文件

这个就是正常的导出了 rules ，而这些 rule 都是来自这个库自己创建的 rule

```js
module.exports = {
  rules: {
    'array-bracket-spacing': require('./rules/array-bracket-spacing'),
    'arrow-spacing': require('./rules/arrow-spacing'),
    'attribute-hyphenation': require('./rules/attribute-hyphenation'),
	}
}
```

可以看到 require 都是本地的路径

那这个正对应了 plugin 可以创建规则那一条

---

下面还有

```js
  configs: {
    base: require('./configs/base'),
    essential: require('./configs/essential'),
    'no-layout-rules': require('./configs/no-layout-rules'),
    recommended: require('./configs/recommended'),
    'strongly-recommended': require('./configs/strongly-recommended'),
    'vue3-essential': require('./configs/vue3-essential'),
    'vue3-recommended': require('./configs/vue3-recommended'),
    'vue3-strongly-recommended': require('./configs/vue3-strongly-recommended')
  },
```

这些配置看看都是啥东西

```js

module.exports = {
  parser: require.resolve('vue-eslint-parser'),
  parserOptions: {
    ecmaVersion: 2020,
    sourceType: 'module'
  },
  env: {
    browser: true,
    es6: true
  },
  plugins: ['vue'],
  rules: {
    'vue/comment-directive': 'error',
    'vue/jsx-uses-vars': 'error'
  }
}

```

原来这个 configs 就是和上面的 eslint-config-xxx 是一样的啊，导出对应的配置文件分享给别人

所以说一个 plugin 是可以包含配置的

---

最后还有一个 processors 选项

```js
  processors: {
    '.vue': require('./processor')
  }
```


在回顾一下 processors 是为了处理非  js 的文件，这个就和 webpack 的 loader 一样

所以这里是为了能让 eslint 检查 SFC 文件的

至此，整个插件的逻辑就分析完了。

那其实到这里 eslint 的生态以及基本开发操作啥的就明白了  

接着就分析分析通过 vue-cli 创建的项目是如何配置的 eslint 吧

这里重要分析 eslint + pretty

## 分析使用 vue-cli 创建项目的 eslint + prettier

直接用 vue-cli 创建一个项目，eslint 选择 prettier

然后我们看看生成的 eslint.config

```js
module.exports = {
  root: true,
  env: {
    node: true
  },
  extends: ["plugin:vue/vue3-essential", "eslint:recommended", "@vue/prettier"],
  parserOptions: {
    parser: "babel-eslint"
  },
  rules: {
    "no-console": process.env.NODE_ENV === "production" ? "warn" : "off",
    "no-debugger": process.env.NODE_ENV === "production" ? "warn" : "off"
  }
};

```

好先一点点的分析

root 指定当前就是根目录了，不需要向上查找别的配置了

env 指定了需要 node 坏境，env 其实就是包含了一系列的全局变量（对全局变量的封装）

parserOptions 指定为 babel-eslint 解析器，是因为需要用到 babel

rules 这里只是自定义处理了两个
	- no-console 
	- no-debugger
	意思都是，如果是在生产坏境下就是 warn 级别 ，否则是 off 
	
	
剩下的 extends 就是重点了，一个一个的来分析

### eslint:recommended

这个是 eslint 官方推荐的规则，那从哪里可以看到这个对应的规则呢？

我在 eslint 源码里面找到了[传送门](https://github.com/eslint/eslint/blob/master/conf/eslint-recommended.js)

里面就是定义了 rule 是什么级别的

### plugin:vue/vue3-essential

那接着看这个，这个 config 是来自 eslint-plugin-vue [传送门](https://github.com/vuejs/eslint-plugin-vue/blob/master/lib/configs/essential.js)

而 essential 翻译过来是至关重要的

可以看到这个配置里面是使用 vue 自定义的 rule

### @vue/prettier

最后一个，来自 eslint-config-prettier [传送门](https://github.com/vuejs/eslint-config-prettier/blob/master/index.js)

为什么需要这个配置呢？ 我们先看看它的源码

入口文件
```js
module.exports = {
  plugins: ['prettier'],
  extends: [
    require.resolve('eslint-config-prettier'),
    require.resolve('eslint-config-prettier/vue')
  ],
  rules: {
    'prettier/prettier': 'warn'
  }
}	
```

首先是应用了 prettier/prettier 的 rules，并且把所有的 rule 设置为 warn 级别
> eslint-plugin-prettier 里面只有一个 prettier 这一个 rule

接着看 eslint-config-prettier [传送门](https://github.com/prettier/eslint-config-prettier/blob/master/index.js)

prettier 的配置其实就是把 eslint 官方关于样式上的 rule 全部关掉，不然的话，prettier 和 eslint 在处理代码的样式上会有冲突

在看 eslint-config-prettier/vue [传送门](https://github.com/prettier/eslint-config-prettier/blob/master/vue.js)

这个配置和上面的是一样的道理，只不过是把 prettier 和 vue 样式相冲突的 rule 全部关掉

那总结一下 @vue/prettier 其实就是为了处理样式冲突的

有两个冲突
- prettier 和 eslint 的样式冲突
- prettier 和 vue 的样式冲突


---

## 总结

好了，至此整个 eslint 就已经分析完了。现在已经很明确的知道 vue 是如何处理 eslint + prettier 了。

其实就是创建了自己的 eslint 检查 rule，然后关闭一些和 prettier 冲突的样式






















