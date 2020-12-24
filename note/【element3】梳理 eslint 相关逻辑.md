## 起因
在开发 element3 的时候发现 eslint 过不去，比如不支持 ?. 这种写法

还有就是样式上的问题，在 mac 和 windows 坏境下还有差异，等等一系列问题吧

然后看了看 发现 element 原有的 eslint 版本太低了 大概在 4.xx ，现在已经是 7.xx 了

索性来个大升级吧

而且正好借这个机会好好的研究研究eslint 的原理


这个参考资料的《深入理解 ESlint》讲解的不错，从历史到原理到插件的开发都有说明

看完基本也就明白 eslint 是怎么个原理了

小步走~~

先把项目集成 eslint

- 先删除了之前所有 eslint 的内容
- 需要要 npm uninstall 删除，npm 会自动删除 lock 文件里面的配置

直接直接使用 npx eslint --init 来创建所有的 eslint 怀集即可

但是我这里选择了 code style ，看看如何集成 prettier 吧

完事之后再去看看如何集成 prettier

那接下来看看 eslint 如何和Prettier 配合一起使用吧

集成prettier 看参考资料就可以，写的是比较全面的，这里就不在赘述了

其实除了这个问题还有一个问题是 eslint 如何在 vscode 中使用？(todo)

这里是直接把之前的 eslint 删除了，然后重新下载了一个最新的，就好了。。。
```js
 "editor.codeActionsOnSave": {
        "source.fixAll.eslint": true
    }
```
多了个这个配置，自动 fix所有的文件，看起来应该是使用 eslint 来修复，而 eslint 用的是 prettier 来美化代码


## 参考资料
- [深入理解 ESlint](https://juejin.im/post/6844903901292920846)
- [使用ESLint+Prettier来统一前端代码风格](https://juejin.im/post/6844903621805473800)

## 原 element eslint 配置项
```js
{
  "env": {
    "jest": true,
    "es6": true
  },
  "globals": {
    "ga": true,
    "chrome": true
  },
  "plugins": ["html", "json"],
  "extends": "elemefe",
  "rules": {
    "no-restricted-globals": ["error", "event", "fdescribe"],
    "semi": [2, "never"]
  },
  "parserOptions": {
    "ecmaVersion": 8,
    "ecmaFeatures": {
      "jsx": true
    }
  }
}



    "eslint": "4.18.2",
    "eslint-config-elemefe": "0.1.1",
    "eslint-loader": "^2.0.0",
    "eslint-plugin-html": "^4.0.1",
    "eslint-plugin-json": "^1.2.0",
```


setting.json 备忘配置

等会研究 eslint - vscode 的时候在处理
```js
   "eslint.options": { 
        "extensions": [
            ".js",
            ".vue",
            ".ts",
            ".tsx"
        ]
    },
    "eslint.validate": [
        {
            "language": "html",
            "autoFix": true
        },
        {
            "language": "vue",
            "autoFix": true
        },
        {
            "language": "javascript",
            "autoFix": true
        },
        {
            "language": "javascriptreact",
            "autoFix": true
        },
        {
            "language": "typescript",
            "autoFix": true
        }
    ],
    "eslint.enable": true,
    "eslint.autoFixOnSave": true,
```