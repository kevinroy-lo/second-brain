## 看得见的思考

先看 Switch 组件的实现逻辑

---

看 setup 的实现

```js
 setup(props, { emit }) {
    let api = inject(GroupContext, null)
    let id = `headlessui-switch-${useId()}`

    function toggle() {
      emit('update:modelValue', !props.modelValue)
    }

    return {
      id,
      el: api?.switchRef,
      labelledby: api?.labelledby,
      describedby: api?.describedby,
      handleClick(event: MouseEvent) {
        event.preventDefault()
        toggle()
      },
      handleKeyUp(event: KeyboardEvent) {
        if (event.key !== Keys.Tab) event.preventDefault()
        if (event.key === Keys.Space) toggle()
      },
      // This is needed so that we can "cancel" the click event when we use the `Enter` key on a button.
      handleKeyPress(event: KeyboardEvent) {
        event.preventDefault()
      },
    }
```

这里的通过 inject 过来的 api 是在 SwitchGroup 中给 provide 的

而这里 return 出去的 el 为什么要赋值是一个  switchRef 呢？  

> 已知 switchRef 是一个 ref 类型的对象

这里的 labelledby 和 describedby 又是什么？

会在哪里用得？

剩下的几个点击和键盘事件就比较好理解了，应该是在后面的 render 里面会用到


---

接下来看  render 函数的实现

```js
 render() {
    let api = inject(GroupContext, null)
    let { class: defaultClass, className = defaultClass } = this.$props

    let slot = { checked: this.$props.modelValue }
    let propsWeControl = {
      id: this.id,
      ref: api === null ? undefined : api.switchRef,
      role: 'switch',
      tabIndex: 0,
      class: resolvePropValue(className, slot),
      'aria-checked': this.$props.modelValue,
      'aria-labelledby': this.labelledby,
      'aria-describedby': this.describedby,
      onClick: this.handleClick,
      onKeyup: this.handleKeyUp,
      onKeypress: this.handleKeyPress,
    }

    if (this.$props.as === 'button') {
      Object.assign(propsWeControl, { type: 'button' })
    }

    return render({
      props: { ...this.$props, ...propsWeControl },
      slot,
      attrs: this.$attrs,
      slots: this.$slots,
      name: 'Switch',
    })
  },
```

这里有一个很关键的信息是 propsWeControl 

整个 render 逻辑其实就是为了创建出来 propsWeControl ，然后这个 propsWeControl 其实是作为组件的 props 来处理的

而后续的关键函数是 render（调用了一个封装好的 render 函数）

还有一个有意思的点事 slot ，这个 slot 其实就是个普通的对象，但是里面包含了这个 Switch 组件的状态

剩下的就是把组件的信息透传给 render 函数，比如 this.$attrs, this.$slots 这些关于外部的信息

那我们的重点就需要放到这个 render 函数的实现上了

---

分析内部的 render 函数

```js
export function render({
  visible = true,
  features = Features.None,
  ...main
}: {
  props: Record<string, any>
  slot: Record<string, any>
  attrs: Record<string, any>
  slots: Slots
  name: string
} & {
  features?: Features
  visible?: boolean
}) {
  // Visible always render
  if (visible) return _render(main)

  if (features & Features.Static) {
    // When the `static` prop is passed as `true`, then the user is in control, thus we don't care about anything else
    if (main.props.static) return _render(main)
  }

  if (features & Features.RenderStrategy) {
    let strategy = main.props.unmount ?? true ? RenderStrategy.Unmount : RenderStrategy.Hidden

    return match(strategy, {
      [RenderStrategy.Unmount]() {
        return null
      },
      [RenderStrategy.Hidden]() {
        return _render({
          ...main,
          props: { ...main.props, hidden: true, style: { display: 'none' } },
        })
      },
    })
  }

  // No features enabled, just render
  return _render(main)
}


```

这个 render 函数就是个处理边界情况的，而最终的核心逻辑都在 _render 函数内


暂时看不太懂这几个 if 的逻辑是为了什么， 那么就先看看 _render 都干了什么吧


---

分析 _render 函数


```js
function _render({
  props,
  attrs,
  slots,
  slot,
  name,
}: {
  props: Record<string, any>
  slot: Record<string, any>
  attrs: Record<string, any>
  slots: Slots
  name: string
}) {
  let { as, ...passThroughProps } = omit(props, ['unmount', 'static'])

  let children = slots.default?.(slot)

  if (as === 'template') {
    if (Object.keys(passThroughProps).length > 0 || Object.keys(attrs).length > 0) {
      let [firstChild, ...other] = children ?? []

      if (!isValidElement(firstChild) || other.length > 0) {
        throw new Error(
          [
            'Passing props on "template"!',
            '',
            `The current component <${name} /> is rendering a "template".`,
            `However we need to passthrough the following props:`,
            Object.keys(passThroughProps)
              .concat(Object.keys(attrs))
              .map(line => `  - ${line}`)
              .join('\n'),
            '',
            'You can apply a few solutions:',
            [
              'Add an `as="..."` prop, to ensure that we render an actual element instead of a "template".',
              'Render a single element as the child so that we can forward the props onto that element.',
            ]
              .map(line => `  - ${line}`)
              .join('\n'),
          ].join('\n')
        )
      }

      return cloneVNode(firstChild, passThroughProps as Record<string, any>)
    }

    if (Array.isArray(children) && children.length === 1) {
      return children[0]
    }

    return children
  }

  return h(as, passThroughProps, children)
}

```

这里的 as 其实就是在 switch setup 里面的 props 的默认值

如果看文档的话， switch  的 as  props 的默认值是 button

我们现在先把 if (as === 'template')  这个逻辑处理干掉

那么其实最终就是做了用 h 函数创建一个 vnode 

而 vnode 的 type 就是 as 的值

以及传过来的 props 和 调用 slots.default?.(slot) 生成的 children

这里有意思的是  slot 的作用（上面提到过）它在这里是以 default slot 的值给入的，

这样的话 用户就可以基于这个 slot 的值（其实就获取到了当前组件的状态）来做不同的事情

----

### 总结

headless 的逻辑是什么

就是创建一个只有逻辑的组件，但是这个组件是需要一个 base element 作为载体的，这里的载体就是用  as 来声明的，最终的这个 as 是这个组件的真实的 dom ，
> as 是可以让用户自定义的

那剩下的就是组件自己维护好自己的内部状态，还需要把外部的状态暴露给用户，暴露的方式通过 slot 的方式以及 class 的方式

用户基于暴露出来的组件外部状态来做 ui 的处理

---

那么我们在让事情复杂点

像多个组件联合起来互相作用的场景是怎么处理

比如在 switch 这里有 switch 和 label 联合起来的功能

点击 label 的时候 switch 会切换状态


---

基于 switch 的话，那么肯定也会有 switchLabel 这个组件

```js
// switch.js 
// 第 130 行代码
export let SwitchLabel = Label
export let SwitchDescription = Description
```

果不其然，switch.js 的 130 代码发现了

看看这个 Label 都是什么逻辑

```js
// label.js
setup() {
    let context = useLabelContext()
    let id = `headlessui-label-${useId()}`

    onMounted(() => onUnmounted(context.register(id)))

    return { id, context }
  },


```

先看 setup ，里面有个 context 这个是个什么东西呢？

看下面的调用的话，是个label 管理器，用id 注册了当前的 label 

这里的使用也有意思 ，先 onMounted 然后再里面用  onUnmounted  这个理解是有问题的

onUnmounted 直接注册就可以了

那问题来了，为什么要在 label 销毁的时候在 register 呢？ 

噢，不对，这个 context.register(id) 应该是一执行 onMounted 的时候就执行了，，

猜测是会返回一个函数用来卸载，所以正好可以作为 onUnmounted 的执行函数

----

分析 label 组件的 render 函数

```js

 render() {
    let { name = 'Label', slot = {}, props = {} } = this.context
    let { passive, ...passThroughProps } = this.$props
    let propsWeControl = {
      ...Object.entries(props).reduce(
        (acc, [key, value]) => Object.assign(acc, { [key]: unref(value) }),
        {}
      ),
      id: this.id,
    }
    let allProps = { ...passThroughProps, ...propsWeControl }

    // @ts-expect-error props are dynamic via context, some components will
    //                  provide an onClick then we can delete it.
    if (passive) delete allProps['onClick']

    return render({
      props: allProps,
      slot,
      attrs: this.$attrs,
      slots: this.$slots,
      name,
    })
  },

```

看 render 函数里面的实现，最关键的信息就是 allProps 的合成了

> 最后调用的 render 函数和 switch 的逻辑是一样的

先从 this.context 中取出 slot 和 props，那么这个 context 在 runtime 的时候它的值是什么？？ 

是什么时候给 context 赋值的？ 这个逻辑还没有看到

而 propsWeControl 的值是 copy 的 this.context.props 加上 label 自己的 id 

所以关键的信息点就是要知道 this.context 到底是什么

最后 allProps 的值就是 passThroughProps （用户传入的 prop 的值和 propsWeControl）

---

来分析分析 this.context 是什么吧

label 中的 context 一开始是通过  useLabelContext 来创建的

所以看 useLabelContext

```js
let LabelContext = Symbol('LabelContext') as InjectionKey<{
  register(value: string): () => void
  slot: Record<string, unknown>
  name: string
  props: Record<string, unknown>
}>

function useLabelContext() {
  let context = inject(LabelContext, null)
  if (context === null) {
    let err = new Error('You used a <Label /> component, but it is not inside a parent.')
    if (Error.captureStackTrace) Error.captureStackTrace(err, useLabelContext)
    throw err
  }
  return context
}

```

是通过 LabelContext 这个 symbol 类型来取出的值，看看这个是在什么时候 provide 的

---

provide 的逻辑在 	useLabels 函数里

```js
// label.ts
// 38 行代码

export function useLabels({
  slot = {},
  name = 'Label',
  props = {},
}: {
  slot?: Record<string, unknown>
  name?: string
  props?: Record<string, unknown>
} = {}): ComputedRef<string | undefined> {
  let labelIds = ref<string[]>([])
  function register(value: string) {
    labelIds.value.push(value)

    return () => {
      let idx = labelIds.value.indexOf(value)
      if (idx === -1) return
      labelIds.value.splice(idx, 1)
    }
  }

  provide(LabelContext, { register, slot, name, props })

  // The actual id's as string or undefined.
  return computed(() => (labelIds.value.length > 0 ? labelIds.value.join(' ') : undefined))
}


```

而 provide 的值，是调用这个 useLabels 的值，比如 props 和 slot 

这里也看到了 register ，果然和刚刚分析的一样。 就是把 label 的id 记录下来，并且返回一个 函数，调用的时候会吧 这个  id 的 label 给移出掉

看看记录这个 labelIds 有什么用呢？？

在最后一句代码里面使用到了，是返回了一个 computed 属性，值是所有 labelids join 之后的值，这个有啥用呀、？？？？

看看在 switch 里面是如何使用 useLabels 这个函数的

---

啊哦，使用 useLabels 这个函数的是  switchGroup 这个组件

那么我们正好可以分析分析

先是 setup

```js

setup(props, { slots, attrs }) {
    let switchRef = ref<StateDefinition['switchRef']['value']>(null)
    let labelledby = useLabels({
      name: 'SwitchLabel',
      props: {
        onClick() {
          if (!switchRef.value) return
          switchRef.value.click()
          switchRef.value.focus({ preventScroll: true })
        },
      },
    })
    let describedby = useDescriptions({ name: 'SwitchDescription' })

    let api = { switchRef, labelledby, describedby }

    provide(GroupContext, api)

    return () => render({ props, slot: {}, slots, attrs, name: 'SwitchGroup' })
  },



```

如果我们暂时只关注 useLabels 的话，那么就明白了 label 的使用

使用的时候通过 useLabels 给注入 props ，这个 props 最终是给到 label 的，所以相当于是给 label 传值

而这个 labelledby 有什么用，暂时还不知道

---

那所以思考一个问题，多个组件是如何联合起来的？

那联合起来的本质是需要获取到组件的实例对象，方便后续可以直接访问到

那我们就拿 switch 和 label 是怎么联合起来的举例

首先必须得有一个容器把这2个组件收集起来，而这个就是 switchGroup 的作用

所以有了 switchRef 和  labelledby

而 useLabel 的作用是给这个 switchGroup 下的所以子组件注入对应的值，而这个值后面是在创建的 label 组件内 inject 获取到的，也就是变相的给传值的

这个得本质就是在 group 中通过这种机制来控制了 label 的内容

接着是在 group 中通过 provide 的方式把值给到所有子组件

在 switch组件中，他先获取了 group 的 switchRef ，给到自己的 ref 上，也就是给 switchRef 赋值。

这样 switchRef 就有值了，那么在 click label 的时候的逻辑，就是通过 switchRef 来获取到的 switch 组件的实例，然后调用的 switch.value.click 函数，作为联合逻辑

本质上就是几个类之间的调用

那么所以，基于 provide 和 inject 来获取组件和组件之间实例，然后把逻辑联合起来。
