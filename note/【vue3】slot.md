研究研究 slot 插槽是怎么实现的

其实我 vue2 的时候也没怎么用过插槽，而且不是太清楚再什么场景下用插槽要不用 props 更好，咱今天就来探一探 slot 的本质，看看是怎么实现的，着有助于我理解 slot 的本质，从而可以用好它


---
## 看的见的思考

我们需要从 template 编译后的结果来入手，看看具体是调用了哪个函数

```js
// tempalte
<div id="app">
   <Comp>
     1233
   </Comp>
</div>

// 编译后
export function render(_ctx, _cache, $props, $setup, $data, $options) {
  const _component_Comp = _resolveComponent("Comp")

  return (_openBlock(), _createBlock("div", { id: "app" }, [
    _createVNode(_component_Comp, null, {
      default: _withCtx(() => [
        _createTextVNode(" 1233 ")
      ]),
      _: 1
    })
  ]))
}
```

这里的插槽被放到了 createVNode 的第三个参数，第三个参数是作为 children 来处理的，所以插槽的本质就是 children 吗？

平时看到的 children 都是 array 类型，这里插槽的话是个对象 。 

我们看看在 createVNode 里面是怎么处理对象的

```js

// _createVNode 函数内

// 在这里处理的 children
normalizeChildren(vnode, children)


```

我们看看 normalizeChildren 函数

```js
// normalizeChildren 函数内

  } else if (typeof children === 'object') {
    // Normalize slot to plain children
    if (
      (shapeFlag & ShapeFlags.ELEMENT || shapeFlag & ShapeFlags.TELEPORT) &&
      (children as any).default
    ) {
      normalizeChildren(vnode, (children as any).default())
      return
    } else {
```

这里的判断有意思，只有是 element 和 teleport 类型并且有 default 字段的话才会处理

哇哦，这里是递归的操作，把 children.default() 内的返回值拿出来继续 normalizeChildren，那我们得看看 这个返回值是个啥

```js
 return (_openBlock(), _createBlock("div", { id: "app" }, [
    _createVNode(_component_Comp, null, {
      default: _withCtx(() => [
        _createTextVNode(" 1233 ")
      ]),
      _: 1
    })
  ]))

```

回顾一下，这里是调用了 withCtx， 那我们看看它都干了啥

```js
/**
 * Wrap a slot function to memoize current rendering instance
 * @private
 */
export function withCtx(
  fn: Slot,
  ctx: ComponentInternalInstance | null = currentRenderingInstance
) {
  if (!ctx) return fn
  return function renderFnWithContext() {
    const owner = currentRenderingInstance
    setCurrentRenderingInstance(ctx)
    const res = fn.apply(null, arguments as any)
    setCurrentRenderingInstance(owner)
    return res
  }
}
```

好吧，暂时也看不出来干啥的 ，但是 memoize 是个缓存结果的意思，先过

这里就是返回了对应的 fn，而 fn 是什么，这里的类型就是个 Slot 类型，看看 Slot 类型是啥样子

```js
export type Slot = (...args: any[]) => VNode[]

```

就是个普通的函数，返回值是个 vnode 数组

那我们再次回顾一下，这个 slot 函数里面是什么呢

```js
  return (_openBlock(), _createBlock("div", { id: "app" }, [
    _createVNode(_component_Comp, null, {
      default: _withCtx(() => [
        _createTextVNode(" 1233 ")
      ]),
      _: 1
    })
  ]))
```

啊哦，果然返回了 vnode  的数组，那如果这么看的话就是普通的 vnode 啊，只不过定义的方式不同

分析到这里就断了，，找不到后续了

这里我搜到了 ComponentSlots.ts， 来看看它吧

---
ComponentSlots

先看测试 componentSlots.spec.ts

这里尽然只有一个测试，而且还有一个 TODO

```js
describe('component: slots', () => {
  // TODO more tests for slots normalization etc.
```

好吧，等我分析完了，可以给添加点测试，测试不全只能先从代码逻辑看起了

回到 componentSlot.ts 内

找到了一个 initSlots 函数，先从它看起

```js
export const initSlots = (
  instance: ComponentInternalInstance,
  children: VNodeNormalizedChildren
) => {
  if (instance.vnode.shapeFlag & ShapeFlags.SLOTS_CHILDREN) {
    const type = (children as RawSlots)._
    if (type) {
      instance.slots = children as InternalSlots
      // make compiler marker non-enumerable
      def(children as InternalSlots, '_', type)
    } else {
      normalizeObjectSlots(children as RawSlots, (instance.slots = {}))
    }
  } else {
    instance.slots = {}
    if (children) {
      normalizeVNodeSlots(instance, children)
    }
  }
  def(instance.slots, InternalObjectKey, 1)
}
```

这里是先看看 vnode 是不是 slots_children ，那么问题来了，是从哪里给赋值为 slots_children 的呢？

找到了 ，是在 normalizeChildren 的时候，我们刚刚少看了

如果 children 是个 object 的话，并且不是 element 和 teleport 类型的话就会触发 else 逻辑
```js
 if (
      (shapeFlag & ShapeFlags.ELEMENT || shapeFlag & ShapeFlags.TELEPORT) &&
      (children as any).default
    ) {
      normalizeChildren(vnode, (children as any).default())
      return
    } else {
      type = ShapeFlags.SLOTS_CHILDREN
      const slotFlag = (children as RawSlots)._
      if (!slotFlag && !(InternalObjectKey in children!)) {
        // if slots are not normalized, attach context instance
        // (compiled / normalized slots already have context)
        ;(children as RawSlots)._ctx = currentRenderingInstance
      } else if (slotFlag === SlotFlags.FORWARDED && currentRenderingInstance) {
        // a child component receives forwarded slots from the parent.
        // its slot type is determined by its parent's slot type.
        if (
          currentRenderingInstance.vnode.patchFlag & PatchFlags.DYNAMIC_SLOTS
        ) {
          ;(children as RawSlots)._ = SlotFlags.DYNAMIC
          vnode.patchFlag |= PatchFlags.DYNAMIC_SLOTS
        } else {
          ;(children as RawSlots)._ = SlotFlags.STABLE
        }
      }
    }
```

就是在这个时候设置为 SLOTS_CHILDREN 类型的

我先看看这个函数是什么时候调用的把

```js
// setupComponent 

  initSlots(instance, children)

```

啊哦，是在 setupComponent 的时候调用的，那我们之前分析过， slot 过来就是 children。接着我们在返回去看 initSlots

```js
// initSlots
 if (vnode.shapeFlag & ShapeFlags.SLOTS_CHILDREN) {
    const type = (children as RawSlots)._
    if (type) {
      // compiled slots.
      if (__DEV__ && isHmrUpdating) {
        // Parent was HMR updated so slot content may have changed.
        // force update slots and mark instance for hmr as well
        extend(slots, children as Slots)
      } else if (type === SlotFlags.STABLE) {
        // compiled AND stable.
        // no need to update, and skip stale slots removal.
        needDeletionCheck = false
      } else {
        // compiled but dynamic (v-if/v-for on slots) - update slots, but skip
        // normalization.
        extend(slots, children as Slots)
      }
    }
```
先只看 if 里面的逻辑，这里还需要看看 RawSlots 是个什么东西

```js
export type RawSlots = {
  [name: string]: unknown
  // manual render fn hint to skip forced children updates
  $stable?: boolean
  /**
   * for tracking slot owner instance. This is attached during
   * normalizeChildren when the component vnode is created.
   * @internal
   */
  _ctx?: ComponentInternalInstance | null
  /**
   * indicates compiler generated slots
   * we use a reserved property instead of a vnode patchFlag because the slots
   * object may be directly passed down to a child component in a manual
   * render function, and the optimization hint need to be on the slot object
   * itself to be preserved.
   * @internal
   */
  _?: SlotFlags
}
```

先看 _ 把，这个是属于 slotFlags 特有的 flags，可以参考一下 vnode 的 flags，他们都是差不多的，应该是区分 slots 的类型的

继续回到 initSlots 内

```js
export const initSlots = (
  instance: ComponentInternalInstance,
  children: VNodeNormalizedChildren
) => {
  if (instance.vnode.shapeFlag & ShapeFlags.SLOTS_CHILDREN) {
    const type = (children as RawSlots)._
    if (type) {
      instance.slots = children as InternalSlots
      // make compiler marker non-enumerable
      def(children as InternalSlots, '_', type)
    } else {
      normalizeObjectSlots(children as RawSlots, (instance.slots = {}))
    }
  } else {
    instance.slots = {}
    if (children) {
      normalizeVNodeSlots(instance, children)
    }
  }
  def(instance.slots, InternalObjectKey, 1)
}
```

这里干了这么几件事

如果 type 有的话，那么设置 instance.slots 为 children ，然后把type 挂载到 children 对象上

如果type 没有的话，直接调用 normalizeObjectSlots 

上面说的是 SLOTS_CHILDREN 类型

下面如果不是的话，那么先 instance.slots = {} ，然后如果有 children 的话，就调用  normalizeVNodeSlots 

接着我们就分析分析  normalizeObjectSlots 以及  normalizeVNodeSlots

```js
const normalizeObjectSlots = (rawSlots: RawSlots, slots: InternalSlots) => {
  const ctx = rawSlots._ctx
  for (const key in rawSlots) {
    if (isInternalKey(key)) continue
    const value = rawSlots[key]
    if (isFunction(value)) {
      slots[key] = normalizeSlot(key, value, ctx)
    } else if (value != null) {
      const normalized = normalizeSlotValue(value)
      slots[key] = () => normalized
    }
  }
}

```


---

插一下，又是洗澡时来的感悟，，

上面这些步骤其实都是为了收集插槽，存在 instance.slots 内，只不过这里是插槽如果是组件的话和非组件的时候处理方式不一样（有待验证）

---

继续继续 先看 normalizeObjectSlots  和 normalizeVNodeSlots

我擦擦，有点复杂，但是总的来说就是 normalize(正常化) slot ,最终转换成标准可渲染的 vnode 。

initSlots 之后 ， instance.slots 内就有值了。接着看看怎么处理 instance.slots 的值的


---

updateSlots

这个函数是在  updateComponentPreRender 时候调用的

```js
 const updateComponentPreRender = (
    instance: ComponentInternalInstance,
    nextVNode: VNode,
    optimized: boolean
  ) => {
    if (__DEV__ && instance.type.__hmrId) {
      optimized = false
    }
    nextVNode.component = instance
    const prevProps = instance.vnode.props
    instance.vnode = nextVNode
    instance.next = null
    updateProps(instance, nextVNode.props, prevProps, optimized)
    updateSlots(instance, nextVNode.children)
  }


```

updateSlots

```js
export const updateSlots = (
  instance: ComponentInternalInstance,
  children: VNodeNormalizedChildren
) => {
  const { vnode, slots } = instance
  let needDeletionCheck = true
  let deletionComparisonTarget = EMPTY_OBJ
  if (vnode.shapeFlag & ShapeFlags.SLOTS_CHILDREN) {
    const type = (children as RawSlots)._
    if (type) {
      // compiled slots.
      if (__DEV__ && isHmrUpdating) {
        // Parent was HMR updated so slot content may have changed.
        // force update slots and mark instance for hmr as well
        extend(slots, children as Slots)
      } else if (type === SlotFlags.STABLE) {
        // compiled AND stable.
        // no need to update, and skip stale slots removal.
        needDeletionCheck = false
      } else {
        // compiled but dynamic (v-if/v-for on slots) - update slots, but skip
        // normalization.
        extend(slots, children as Slots)
      }
    } else {
      needDeletionCheck = !(children as RawSlots).$stable
      normalizeObjectSlots(children as RawSlots, slots)
    }
    deletionComparisonTarget = children as RawSlots
  } else if (children) {
    // non slot object children (direct value) passed to a component
    normalizeVNodeSlots(instance, children)
    deletionComparisonTarget = { default: 1 }
  }

  // delete stale slots
  if (needDeletionCheck) {
    for (const key in slots) {
      if (!isInternalKey(key) && !(key in deletionComparisonTarget)) {
        delete slots[key]
      }
    }
  }
}


```

不好意思，没怎么看懂，代码看起来就是要最终删除一些过期的 slots ，但是整个流程没有串起来呀。。

看来我得找点 test 来看看

奶奶的，test 也没有几个， 那 slot 先放一放，让 R脑 思考思考

---

再次看 slot 的实现

先来从使用的角度来看

Component 组件类型下的 children 就是 slot 了
而组件在初始化的时候，会把 children render 出来，而 slot 接收的是 object，而我们知道 children 必须是 array 类型，
并且需要把 slots 存储到 instance.slots 对象内
其实还有一个点，是组件必须定义了 slot 才可以渲染

这里总结一下：
- component slot（children）需要规范一下格式
- 需要把 slot 存储到 instance.slots 内
- 当前的这个 slots 要不要渲染（只有加入到 children 内了，那么就是可以渲染的）

那么基于以上几点，我们在重新看看源码


```js
export const initSlots = (
  instance: ComponentInternalInstance,
  children: VNodeNormalizedChildren
) => {
  // 是在哪里初始化 slots_children 这个 flag 的
  if (instance.vnode.shapeFlag & ShapeFlags.SLOTS_CHILDREN) {
    const type = (children as RawSlots)._
    if (type) {
      instance.slots = children as InternalSlots
      // make compiler marker non-enumerable
      def(children as InternalSlots, '_', type)
    } else {
      normalizeObjectSlots(children as RawSlots, (instance.slots = {}))
    }
  } else {
    instance.slots = {}
    if (children) {
      normalizeVNodeSlots(instance, children)
    }
  }
  def(instance.slots, InternalObjectKey, 1)
}
```

先看 else 里面的逻辑，最重要的就是 normalizeVNodeSlots 了，也就是我们上面谈到的规范化 slots

```js

const normalizeVNodeSlots = (
  instance: ComponentInternalInstance,
  children: VNodeNormalizedChildren
) => {
  const normalized = normalizeSlotValue(children)
  instance.slots.default = () => normalized
}

```

因为这个 else 逻辑其实是只有一个 slot 的情况，所以这里是直接赋值给了 instance.slots.default  ，这里其实也就是我们谈到的第二点，给 instance.slots 赋值

> 至于是怎么 normalize 的，这里先不需要关心

看逻辑的话，只有对应的 initSlot 逻辑，并没有其他的逻辑做处理了

那基于编译后生成的代码来看的话，一个组件的 children 可以是一个 object，里面有个默认值是 default ，那么也就是说 render 的时候 children 是允许是 object？ 还是说在 render 之前会对 children 做规范化操作？

```js
// 编译后
export function render(_ctx, _cache, $props, $setup, $data, $options) {
  const _component_Comp = _resolveComponent("Comp")

  return (_openBlock(), _createBlock("div", { id: "app" }, [
    _createVNode(_component_Comp, null, {
      default: _withCtx(() => [
        _createTextVNode(" 1233 ")
      ]),
      _: 1
    })
  ]))
}
```


那么我要带着这个疑问去看看在哪里处理的 children. default  的这个操作

果然，我在 vnode.ts 中找到了处理 children.default 的这个逻辑
```js
export function normalizeChildren(vnode: VNode, children: unknown) {
  let type = 0
  const { shapeFlag } = vnode
  if (children == null) {
    children = null
  } else if (isArray(children)) {
    type = ShapeFlags.ARRAY_CHILDREN
  } else if (typeof children === 'object') {
    if (shapeFlag & ShapeFlags.ELEMENT || shapeFlag & ShapeFlags.TELEPORT) {
      // Normalize slot to plain children for plain element and Teleport
      const slot = (children as any).default
      if (slot) {
        // _c marker is added by withCtx() indicating this is a compiled slot
        slot._c && setCompiledSlotRendering(1)
        normalizeChildren(vnode, slot())
        slot._c && setCompiledSlotRendering(-1)
      }
      return
    } else {
      type = ShapeFlags.SLOTS_CHILDREN
      const slotFlag = (children as RawSlots)._
      if (!slotFlag && !(InternalObjectKey in children!)) {
        // if slots are not normalized, attach context instance
        // (compiled / normalized slots already have context)
        ;(children as RawSlots)._ctx = currentRenderingInstance
      } else if (slotFlag === SlotFlags.FORWARDED && currentRenderingInstance) {
        // a child component receives forwarded slots from the parent.
        // its slot type is determined by its parent's slot type.
        if (
          currentRenderingInstance.vnode.patchFlag & PatchFlags.DYNAMIC_SLOTS
        ) {
          ;(children as RawSlots)._ = SlotFlags.DYNAMIC
          vnode.patchFlag |= PatchFlags.DYNAMIC_SLOTS
        } else {
          ;(children as RawSlots)._ = SlotFlags.STABLE
        }
      }
    }
  } else if (isFunction(children)) {
    children = { default: children, _ctx: currentRenderingInstance }
    type = ShapeFlags.SLOTS_CHILDREN
  } else {
    children = String(children)
    // force teleport children to array so it can be moved around
    if (shapeFlag & ShapeFlags.TELEPORT) {
      type = ShapeFlags.ARRAY_CHILDREN
      children = [createTextVNode(children as string)]
    } else {
      type = ShapeFlags.TEXT_CHILDREN
    }
  }
  vnode.children = children as VNodeNormalizedChildren
  vnode.shapeFlag |= type
}
```

代码有点多，但是我们现在可以只关心两个逻辑点
- 是如何处理 default 的操作的
- 是在什么时候调用的 normalizeChildren


```js
type = ShapeFlags.SLOTS_CHILDREN
      const slotFlag = (children as RawSlots)._
      if (!slotFlag && !(InternalObjectKey in children!)) {
        // if slots are not normalized, attach context instance
        // (compiled / normalized slots already have context)
        ;(children as RawSlots)._ctx = currentRenderingInstance
      } else if (slotFlag === SlotFlags.FORWARDED && currentRenderingInstance) {
        // a child component receives forwarded slots from the parent.
        // its slot type is determined by its parent's slot type.
        if (
          currentRenderingInstance.vnode.patchFlag & PatchFlags.DYNAMIC_SLOTS
        ) {
          ;(children as RawSlots)._ = SlotFlags.DYNAMIC
          vnode.patchFlag |= PatchFlags.DYNAMIC_SLOTS
        } else {
          ;(children as RawSlots)._ = SlotFlags.STABLE
        }
      }
```

这里要不就是给 chidlren._ctx 赋值 ，要不就是对 children._ 赋值，

基于把 vnode.patchFlag 做了一个更改，

那么也就是说，在后面的操作是利用了 children._  或者是 chidlren._ctx 做了一定的处理

通过全局搜索 RawSlots 发现了一个 renderSlot.ts 文件

这个文件应该是重点了

TODO -> 分析 renderSlot.ts 文件逻辑

终于找到了，是在什么时候用的 instance.slots 的了，并且是怎么给转换成 children 的了

一切的答案都在这里

```js

// template
<div\>

<slot\></slot\>

</div\>

// 编译后
export function render(_ctx, _cache, $props, $setup, $data, $options) {

	return (_openBlock(), _createBlock("div", null, [

	_renderSlot(_ctx.$slots, "default")
]))
}

```

可以看到，如果一个组件内只有声明了 slot 之后，才会调用到 renderSlot 这个函数
	
而这个函数干了什么


```js
export function renderSlot(
  slots: Slots,
  name: string,
  props: Data = {},
  // this is not a user-facing function, so the fallback is always generated by
  // the compiler and guaranteed to be a function returning an array
  fallback?: () => VNodeArrayChildren
): VNode {
  let slot = slots[name]

  // a compiled slot disables block tracking by default to avoid manual
  // invocation interfering with template-based block tracking, but in
  // `renderSlot` we can be sure that it's template-based so we can force
  // enable it.
  isRenderingCompiledSlot++
  openBlock()
  const validSlotContent = slot && ensureValidVNode(slot(props))
  const rendered = createBlock(
    Fragment,
    { key: props.key || `_${name}` },
    validSlotContent || (fallback ? fallback() : []),
    validSlotContent && (slots as RawSlots)._ === SlotFlags.STABLE
      ? PatchFlags.STABLE_FRAGMENT
      : PatchFlags.BAIL
  )
  isRenderingCompiledSlot--
  return rendered
}

```

可以发现，这里是从 slots 中取的数据，而最终这个函数返回的是一个  vnode

而结合上面编译的结果来看，最终这个 vnode 会添加到 createBlock 中

所以，最关键的信息就是在 initSlot 的时候我们只是存储信息， 把数据放到 instance.slots 中，然后当组件有 slot 的时候，才从 Instance.slots 中拿到具体的数据，给到 createBlock 中，最终渲染出来


---

todo ， 好，这个流程其实已经出来了，那么剩下的就是把所有的逻辑用脑图整理出来

	
	

