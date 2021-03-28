
1. 使用 ref 类型，让逻辑更容易解耦
2. useXXX 一概返回对象，效果如同 export const 
	1. 如果直接返回一个值的话，如同 export default ，在命名上做不到统一+提示
3. 一个 useXXX 相当于是一个功能集合，可以单独对其测试
4. 如果这个 useXXX 涉及到一些组件生命周期的逻辑，那么需要用 vtu 的 function 写法对其测试（todo）
	```js
	// eslint-disable-next-line no-unused-vars
	import { mount } from "@vue/test-utils";
	import { useMouseMove } from "../../src/useMouseMove";

	function mountUse(setup, opts) {
	  const Comp = {
		render() {},
		setup,
	  };

	  return mount(Comp, opts);
	}

	describe("HelloWorld.vue", () => {
	  it("useMouseMove", () => {

		let xValue;
		mountUse(() => {
		  xValue = useMouseMove();
		});

		console.log(xValue);
	  });
	});
	
	```


### 流程
1. 先写 happy path test cases
2. 想要实现这个 happy path 通常还需要很多逻辑
	1. 这些是通过 N 个 useXXX 来支撑的，换句话说，多个 useXXX 组合成了基本的组件功能
	2. 正因为 useXXX 代表这一个功能，所以我们可以对其单独测试
	3. 当所有的 useXXX 测试通过后，功能逻辑理论上也应该是通过的
	
### 理论支撑
- 组合的思想
	- 在 vue-next 中深有体验
- useXXX 是一个完备的功能集合
	- 所以 useXXX 是可以单独测试的
- 而多个 useXXX 组合起来就可以完成复杂的逻辑


### todo
- 需要把上面的流程脱敏(用在 table 上)