

下面会记录每个插件是做什么的

## eslint-plugin-vue

是vue 的官网的 eslint 插件，可以帮助我们快速的发现问题，比如解析 .vue 文件


## @vue/eslint-config-typescript

专门为了 vue ts 的项目准备的 eslint 规则


## TypeScript ESLint Parser

An ESLint parser which leverages [TypeScript ESTree](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/typescript-estree) to allow for ESLint to lint TypeScript source code.

一个 parser 专门去解析 ts 的


## @typescript-eslint/eslint-plugin
An ESLint plugin which provides lint rules for TypeScript codebases.

提供了基于 ts 的规则


## 添加 prettier

- prettier
- eslint-plugin-prettier
- @vue/eslint-config-prettier


### eslint-plugin-prettier

专门给 eslint 使用的 prettier 插件

prettier 专注于样式

而 eslint 专注于 rules

各司其职

但是 eslint 和 prettier 会有冲突的地方，需要借助一些其他的插件来解决 比如 [eslint-config-prettier](https://github.com/prettier/eslint-config-prettier)

### @vue/eslint-config-prettier

专门给 vue-cli 用的，但是它帮助我们解决了 eslint 和 prettier 的冲突问题

其内部就是封装了 eslint-config-prettier



