
## what
分析组件的更新逻辑

看看如果涉及到一个组件级别的更新的话，逻辑是怎么样子的

比如一个 value 值是通过 component props 传入的，那么当这个value 值变更的时候他的逻辑是如何执行的

## 看的见的思考

基于 mini-vue/ 的 [componentUpdate](https://github.com/cuixiaorui/mini-vue/tree/master/example/componentUpdate) 这个例子来研究

逻辑触发点是给 component props 的值发生改变了

```js

export default {
  name: "App",
  setup() {
    const msg = ref("123");

    const changeChildProps = () => {
      msg.value = "456";
    };

    return { msg, changeChildProps };
  },

  render() {
    return h("div", {}, [
      h("div", {}, "你好"),
      h(
        "button",
        {
          onClick: this.changeChildProps,
        },
        "change child props"
      ),
      h(Child, {
        msg: this.msg,
      }),
    ]);
  },
};

```

> 就是这个 msg 

那么这时候首先会触发当前组件 App 的 update 逻辑

```js

function setupRenderEffect(instance, container) {
  instance.update = effect(
    function componentEffect() {
	……

```

因为当前的这个 app 有 3 个子组件，逻辑会遍历当前组件的所有子组件依次对比

看看是不是需要更新，当对比发生在子组件 Child.vue 的时候，我们就需要更新组件的，因为这涉及到需要更新 Child 组件的 props 

```js

// 组件的更新
function updateComponent(n1, n2, container) {
  console.log("更新组件", n1, n2);
  // 更新组件实例引用
  const instance = (n2.component = n1.component);
  // 先看看这个组件是否应该更新
  if (shouldUpdateComponent(n1, n2)) {
    console.log(`组件需要更新: ${instance}`);
    // 那么 next 就是新的 vnode 了（也就是 n2）
    instance.next = n2;
    // 这里的 update 是在 setupRenderEffect 里面初始化的，update 函数除了当内部的响应式对象发生改变的时候会调用
    // 还可以直接主动的调用(这是属于 effect 的特性)
    // 调用 update 再次更新调用 patch 逻辑
    // 在update 中调用的 next 就变成了 n2了
    // ps：可以详细的看看 update 中 next 的应用
    // TODO 需要在 update 中处理支持 next 的逻辑
    instance.update();
  } else {
    console.log(`组件不需要更新: ${instance}`);
    // 不需要更新的话，那么只需要覆盖下面的属性即可
    n2.component = n1.component;
    n2.el = n1.el;
    instance.vnode = n2;
  }
}
```


这里会先看看当前的这个组件是不是需要更新的，这一部分逻辑就是在 shouldUpdateComponent 里面判断的

> shouldUpdateComponent 的逻辑可以看mini-vue源码里面的注释来理解

当发现当前组件需要更新的话， 需要调用 instance.update，这个 update 的赋值逻辑是在 setupRenderEffect 中处理的

接着看 instance.update 里面处理 update 的逻辑

```js

      } else {
        // updateComponent
        // This is triggered by mutation of component's own state (next: null)
        // OR parent calling processComponent (next: VNode)
        let { next, bu, u, parent, vnode } = instance
        let originNext = next
        let vnodeHook: VNodeHook | null | undefined
		
        if (next) {
          next.el = vnode.el
          updateComponentPreRender(instance, next, optimized)
        } else {
          next = vnode
        }
```

注意这个 next 逻辑处理，如果说 next 有值的话， 那么就调用  updateComponentPreRender 来去更新组件的数据

这个包括了 props  slots 等专属于组件的数据

这个逻辑处理完成后，还是会继续对比当前组件的所有子元素的

调用当前组件的 render 函数，获取到最新的 subTree 来继续对比子元素

那么基于这个逻辑的话，只要当然的组件触发了 render 函数，那么他就会从上到下依次来判断是不是需要更新，正因为是单向数据流，只会从上到下，所以就不需要检查父级组件是不是需要更新啦

而且组件会把检测逻辑分层，如果子元素是组件，并且这个组件是不需要更新的话，那么对于这个分支（一个组件就是一个 tree 的分支）就不会继续检查下去了

---


分析

updateComponentPreRender

