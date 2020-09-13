
## 结果
妈的，放弃了。暂时没有很好的解决方案（尤大不给提供 hook）	
如果强加这个功能支持的话，会比较复杂
复杂在没有办法 wrap functional component 的 emit 
因为当调用 functional component 的 render 的时候传过来的 emit 只是一个函数
所以你没有办法去修改 instance.emit 来做到 wrap 
只做的只能是把 functional component 包装一层
但是不调用 fucnational compnent 的 render 的话，并不能知道当前的 component 就是个 functional
先放弃。。。
先修别的 bug 向现实低头



## 问题跟进
- [emitted cannot capture events for for functional components](https://github.com/vuejs/vue-test-utils-next/issues/158)
	- 本来想着从 mount 调用 functioal component 时，给 wrapper 一下，但是发现坑有点多，现在已知的问题是，所有的 functional component 组件都用不了 emit
	- 怎么解决呢？
	- 可以遍历所有 functional component 来 wrapper emit
	- events 存储的地方也需要更改
		- 因为如果是 functional component 的话，是没有 proxy 属性的
		- 所以需要改变 events 存储的位置，来支持 functional component 
		- 可以考虑挂在到 emit 函数上
		- 也可以考虑使用闭包来存储 events
			- 使用闭包可能是最简单的逻辑处理方案
				- 因为不需要考虑把 emit 挂载到哪个组件上
					- 使用 mount 的时候：如果是 functional component 的话是会给添加一个 options component 的。所以触发时获取 events 时是有问题的

  - 还有一个 bug
	  - 如果一个组件树内有一个 functional component 的话，在 functional component 内触发 emit ，也是没有办法收集到信息的。
	  - 这是因为没有触发 wrapperEmit
	  - 现在的方案是可以自己写个 defineComponent 来创建
	  - 如果是在 VTU 内支持的话，需要遍历所有的 functional component 用一个函数 wrapper ，在这个 wrapper 内来处理 emit 


- 方案
	- 用闭包方案存储 events  + 遍历所有的 functional component 类型的组件
		- 这个方案不行，因为所有的 functional component 在 mount 的时候都会被触发
		- 是没有合适的时机去 wrap emit 的
		
		
		
	