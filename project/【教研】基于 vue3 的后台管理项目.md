
## 项目搭建
- [目录](https://juejin.cn/post/6931234988281036807?utm_source=gold_browser_extension)


- [ ] 初始化搭建
	- 使用 vite 创建项目 
		- yarn create @vitejs/app store-admin --template vue-ts
	- 支持 ts  
		1.  Install and add `@vuedx/typescript-plugin-vue` to the [plugins section](https://www.typescriptlang.org/tsconfig#plugins "https://www.typescriptlang.org/tsconfig#plugins") in `tsconfig.json`	
		2.  Delete `src/shims-vue.d.ts` as it is no longer needed to provide module info to Typescript	
		3.  Open `src/main.ts` in VSCode	
		4.  Open the VSCode command palette 5. Search and run "Select TypeScript version" -> "Use workspace version"	
		
		结论： 第四部找不到 这个 “select typescript version” 这个暂时放弃
		
		最新： 换成使用 volar 了
	- 基建
		- eslint
			- prettier
			- 配上 lint-staged 在  git ci 的时候检测提交的 code
			- 配置 .eslintignore 忽略 src/shims-vue.d.ts 
			- lint css 
			- [参考](https://intellisense.dev/post/vite-vue3-typescript/)
			
		- test
		
			- unit
				- vtu 
				- jest
				- 需要指定 test:unit 的路径
				-  testMatch: ['**/tests/unit/**/*.[jt]s?(x)'],
				-  用于和 e2e 测试分离开
				[[集成最新的 VUT]]
			- cypress
				- yarn add cypress --dev
				- 配置 script ->     "test:e2e":"cypress open"
				- 在 integration 里面写测试即可 simple_spec.js
				- 直接在 integration 里面写 xxx.ts 即可（最新的版本是支持 ts 的，不需要在看下面的 fix 方案了）
				- 然后 ts 报错找不到 cy ，需要在 tsconfig 里面 的 types 加上 cypress
				- 然后在 includes 里面加上 cypress 测试的路径
				```js
				//tsconfig.js
				   "types": ["vite/client", "jest", "cypress"],
				   "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue","cypress/integration/*.ts"]
				```
				- 可以基于 vue-cli 的目录组织方式把 cypress 的文件放到 tests/e2e 下面，但是需要更改 cypress.json 
				- cypress.json -》   "pluginsFile": "tests/e2e/plugins/index.ts"
				- 然后所有的文件都可以替换成 ts 

				- [参考资料](https://docs.cypress.io/guides/getting-started/installing-cypress.html)
				- [详细的fix方案](https://glebbahmutov.com/blog/use-typescript-with-cypress/) 
					- 这个方式太麻烦了，还需要配置 webpack ，看看 vue-cli 的解决方案是什么
			- https://mswjs.io/ 
		- git commit check
			- validate-commit-msg
			- yorkie 或者是 husky
			- 在配置个 config
		```js
		// 在 commit 之前调用 vcm
		  "gitHooks": {
  			  "pre-commit": "validate-commit-msg"
  			},
		```
		可参考：[阮一峰的](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html) 
		- vuex
		- vue-router
		- css pre-processors 
			- 写现代的 css 用 postcss-preset-env 来做转换
			- scss( 默认就支持)
		- CI/CD 
		    - 增加 lint
			- 可以参考 tdd 的那个项目
			- 需要修改 test unit 的路径
				- 因为 e2e 和 test unit 的测试放在一起了
				- 通过 设置 jest 的 testMatch
			- 需要让 cypress 跑在非浏览器下
				- cypress run
				- 需要在加一个 script -> test:e2e:ci
			- 配置 github action -> .github/workflows/test.yml
			- 还需要配合 cypress-io/github-action@v2 , 在 ci 坏境打开服务
		- vite 配置 alias
			- ts 也需要配置一下
```js
"paths": {
      "@/*": ["src/*"]
    }
```
		
```js
		resolve: {
  		  alias: [
  		    {
  		      find: '@/',
  		      replacement: join(__dirname, 'src/'),
  		    },
  		  ],
  		},
```




## tasking

- [[vite 集成 eslint]]

## 参考

- [众邦科技-开源商城](https://gitee.com/ZhongBangKeJi/CRMEB-Min/tree/master/src/template/admin)