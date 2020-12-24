## 问题
- @rollup/plugin-replace 是做什么的？

## 正文

今天来看看探一探 componentEmits 属性都干了点啥

因为遇到个 bug 被这个 emits 给坑了

	如果不给组件设置 emits 这个属性的话  

	外部调用 @click 的话，会直接给 el 添加事件

	刚刚没有设置 emits 

	所以内部发出了一个 emit("click")

	外部又给注册了一个 click 时间给 el

	所以造成了两次
	
	当给 emits 设置了 click 之后就不会出现两次了
	
咱就要探一探这个逻辑是怎么实现的

---
先找到对应的测试文件 componentEmits.spec.ts

所有的测试看完，并没有得到一个很清晰的认识

那么我去找找对应的文档先看看

[传送门](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0030-emits-option.md)

这里有详细的文档，先看看文档吧，文档看完之后再去看源码里面它是如何实现的

emits 的出现就是为了明确组件要 emit 的事件，让其他人看到这个组件的时候就明确知道这个组件会发出什么事件，并且还可以校验发出的 payload

- Listener Fallthrough Control 着一点重要了，就是解释了文章开头说到的那个内容，如果在 emits 里面声明了 events ，那么这个 events 就不会被当做原生的侦听器了
	- 哇哦，找到了相关的 rfc [0031-attr-fallthrough.md
](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0031-attr-fallthrough.md)
	- 这篇文章记录了 attr-fallthrough 的事情
	- 如果发现行为不一样的话，可以在回过头来回顾一下这篇文章
	- 所有改动的行为都可以看 attr-fallthrough 这篇文章，都有详细的记录

好了，文档看完了，先看看 emits 的源码实现吧

这一步怎么处理的，我还是很好奇的

--- 
直接打开 componentEmits.ts 

直接从 emit 函数 开始看

如果说 emit 函数内没有判断处理这个事件是来自原生还是自定义的话，那么我想这个逻辑应该是在 initProps 的时候处理的

```js
export function emit(
  instance: ComponentInternalInstance,
  event: string,
  ...args: any[]
) {
  const props = instance.vnode.props || EMPTY_OBJ

  if (__DEV__) {
    const options = normalizeEmitsOptions(instance.type)
    if (options) {
      if (!(event in options)) {
        const propsOptions = normalizePropsOptions(instance.type)[0]
        if (!propsOptions || !(`on` + capitalize(event) in propsOptions)) {
          warn(
            `Component emitted event "${event}" but it is neither declared in ` +
              `the emits option nor as an "on${capitalize(event)}" prop.`
          )
        }
      } else {
        const validator = options[event]
        if (isFunction(validator)) {
          const isValid = validator(...args)
          if (!isValid) {
            warn(
              `Invalid event arguments: event validation failed for event "${event}".`
            )
          }
        }
      }
    }
  }

  let handlerName = `on${capitalize(event)}`
  let handler = props[handlerName]
  // for v-model update:xxx events, also trigger kebab-case equivalent
  // for props passed via kebab-case
  if (!handler && event.startsWith('update:')) {
    handlerName = `on${capitalize(hyphenate(event))}`
    handler = props[handlerName]
  }
  if (!handler) {
    handler = props[handlerName + `Once`]
    if (!instance.emitted) {
      ;(instance.emitted = {} as Record<string, boolean>)[handlerName] = true
    } else if (instance.emitted[handlerName]) {
      return
    }
  }
  if (handler) {
    callWithAsyncErrorHandling(
      handler,
      instance,
      ErrorCodes.COMPONENT_EVENT_HANDLER,
      args
    )
  }
}

```

首先取到 vnode.props 里面的值，这里有意思的是 如果没有的话，给一个空对象

接着是先调用了 normalizeEmitsOptions  而参数是 instance.type (这个 type 如果是组件的话，存的是 组件的 options 也就是用户在外面定义的 options)

看看都是怎么 normalize 的

```js
function normalizeEmitsOptions(
  comp: Component
): ObjectEmitsOptions | undefined {
  if (hasOwn(comp, '__emits')) {
    return comp.__emits
  }

  const raw = comp.emits
  let normalized: ObjectEmitsOptions = {}

  // apply mixin/extends props
  let hasExtends = false
  if (__FEATURE_OPTIONS_API__ && !isFunction(comp)) {
    if (comp.extends) {
      hasExtends = true
      extend(normalized, normalizeEmitsOptions(comp.extends))
    }
    if (comp.mixins) {
      hasExtends = true
      comp.mixins.forEach(m => extend(normalized, normalizeEmitsOptions(m)))
    }
  }

  if (!raw && !hasExtends) {
    return (comp.__emits = undefined)
  }

  if (isArray(raw)) {
    raw.forEach(key => (normalized[key] = null))
  } else {
    extend(normalized, raw)
  }
  return (comp.__emits = normalized)
}

```

先看看这个组件有没有 __emits 有的话，直接就返回了，那看起来 __emits 存了点什么东西呀

声明了一个 normalize 对象，看下面的内容的话，__emits 里面存的东西就是这个 normalize 对象

这里主要处理 apply mixin/extends props  看看有没有 mixin 的 或者是 extends 的props

还有一个很有意思的点是 __FEATURE_OPTIONS_API__

全局搜了一下这个变量应该是通过 rollup 里面的 @rollup/plugin-replace 插件来搞的，

先简单的猜一猜，应该是打包的时候把这个 变量给替换掉的把？ 

先记录一下这个问题

这里没有判断这个 comp 会不会是一个 stirng 

哦对，这个函数是给用户来调用的，而这个函数只有在是组件的时候才会有

所以不需要关心 comp 是不是一个 string

接着看，

```js
  // apply mixin/extends props
  let hasExtends = false
  if (__FEATURE_OPTIONS_API__ && !isFunction(comp)) {
    if (comp.extends) {
      hasExtends = true
      extend(normalized, normalizeEmitsOptions(comp.extends))
    }
    if (comp.mixins) {
      hasExtends = true
      comp.mixins.forEach(m => extend(normalized, normalizeEmitsOptions(m)))
    }
  }
```

如果有 extends 的话，就说明是需要处理的，这里也比较有趣

是递归的调用 normalizeEmitsOptions

这里先看看 extend 的行为是什么？ 

```js
export const extend = Object.assign

```

好吧，这里就是把后面的内容合并到前面的对象里，

因为 extends 肯定是一个对象，而mixins 是一个数组吗？ 我忘记了

去看看官方文档的

```
mixins
类型：Array<Object>

详细：

mixins 选项接收一个混入对象的数组。这些混入对象可以像正常的实例对象一样包含实例选项，这些选项将会被合并到最终的选项中，使用的是和 Vue.extend() 一样的选项合并逻辑。也就是说，如果你的混入包含一个 created 钩子，而创建组件本身也有一个，那么两个函数都会被调用。

Mixin 钩子按照传入顺序依次调用，并在调用组件自身的钩子之前被调用。


```
	
是的，是个数组，数组里面的值都是 component 的 options 

那想一想，和当前组件相关的除了自身，也就是 extends 和 mixins 了。

这里重点要关注这个点

```js

// normalizeEmitsOptions
  if (isArray(raw)) {
    raw.forEach(key => (normalized[key] = null))
  } else {
    extend(normalized, raw)
  }
  return (comp.__emits = normalized)


```

最终 return 出去的是 normalized 的值

而这里对这个值做处理的只有两个点

1. raw 是数组的话，那么normalized 和 raw 里面key 对应的值都赋值成  null
2. raw 不是数组，这里有可能是函数，那么直接把 raw 里面的值合并到 normalized 内

这里要说明一下 raw 是什么

```js
 const raw = comp.emits

```
也就是vue3 新增的 emits 属性

好了，其实看到这里还是有一点点迷糊，但是肯定的是这里把所有的 emits 都赋值给了 comp.__emits 上，接着看看要怎么用吧

明白了，这里其实 normalized 的作用就是收集当前组件所有的 emits 信息

而 extends 和 mixins 属性 都有可能是递归的

比如 extends 一个配置，而这个配置里面又有一个 extends 

mixins 也是一样的，所以要用递归来获取所有的 emits 数据

这里就需要回到 emit 函数内了

```js
 if (__DEV__) {
    const options = normalizeEmitsOptions(instance.type)
    if (options) {
      if (!(event in options)) {
        const propsOptions = normalizePropsOptions(instance.type)[0]
        if (!propsOptions || !(`on` + capitalize(event) in propsOptions)) {
          warn(
            `Component emitted event "${event}" but it is neither declared in ` +
              `the emits option nor as an "on${capitalize(event)}" prop.`
          )
        }
      } else {
        const validator = options[event]
        if (isFunction(validator)) {
          const isValid = validator(...args)
          if (!isValid) {
            warn(
              `Invalid event arguments: event validation failed for event "${event}".`
            )
          }
        }
      }
    }
  }
```

这里的 options 就是我们之前处理完的 __emits ，看看都干了点啥吧

这里还有一个大函数 normalizePropsOptions 先给它忽略掉

先看逻辑

如果说要发出的 event 不在 options 里面的话，就给出一个警告

```js
  if (!(event in options)) {
        const propsOptions = normalizePropsOptions(instance.type)[0]
        if (!propsOptions || !(`on` + capitalize(event) in propsOptions)) {
          warn(
            `Component emitted event "${event}" but it is neither declared in ` +
              `the emits option nor as an "on${capitalize(event)}" prop.`
          )
        }
```

因为 options 是已经处理完的，包含当前组件相关的所有 emits 配置

```js
      } else {
        const validator = options[event]
        if (isFunction(validator)) {
          const isValid = validator(...args)
          if (!isValid) {
            warn(
              `Invalid event arguments: event validation failed for event "${event}".`
            )
          }
        }
      }
```
这里就是如果发现是个函数的话，那么肯定就是 validator ，执行一下 看看是不是返回 true，如果不是的话，就给个警告

综上就是检测，如果没有定义 emits 的话，那么就给个警告，如果emits 的值是个函数的话就说明是个 validate ，那么就检测一下

在接着看

```js
  let handlerName = `on${capitalize(event)}`
  let handler = props[handlerName]
  // for v-model update:xxx events, also trigger kebab-case equivalent
  // for props passed via kebab-case
  if (!handler && event.startsWith('update:')) {
    handlerName = `on${capitalize(hyphenate(event))}`
    handler = props[handlerName]
  }
  if (!handler) {
    handler = props[handlerName + `Once`]
    if (!instance.emitted) {
      ;(instance.emitted = {} as Record<string, boolean>)[handlerName] = true
    } else if (instance.emitted[handlerName]) {
      return
    }
  }
  if (handler) {
    callWithAsyncErrorHandling(
      handler,
      instance,
      ErrorCodes.COMPONENT_EVENT_HANDLER,
      args
    )
  }
```

首先是把 event 转成 onEventName 

然后看看在 props 里有没有对应的 handler

这里要注意的是如果是 v-model 的话，需要单独在处理一下

```js
  // for v-model update:xxx events, also trigger kebab-case equivalent
  // for props passed via kebab-case
  if (!handler && event.startsWith('update:')) {
    handlerName = `on${capitalize(hyphenate(event))}`
    handler = props[handlerName]
  }

```

这里还处理了只监听一次的情况，也就是后缀为 Once 的情况

而 emitted 也是为了处理只监听一次的情况，记录一下
```js
  if (!handler) {
    handler = props[handlerName + `Once`]
    if (!instance.emitted) {
      ;(instance.emitted = {} as Record<string, boolean>)[handlerName] = true
    } else if (instance.emitted[handlerName]) {
      return
    }
  }
```

最后就是调用对应的 handler 了
```js
  if (handler) {
    callWithAsyncErrorHandling(
      handler,
      instance,
      ErrorCodes.COMPONENT_EVENT_HANDLER,
      args
    )
  }
```

啊哦，果然这里没有处理如果在 emits 里面声明好了的话，原生的事件侦听就不会调用了

那只能去 initProps 里面找找答案了

---
打开 componentProps.ts 

找到 initProps 函数

```js
export function initProps(
  instance: ComponentInternalInstance,
  rawProps: Data | null,
  isStateful: number, // result of bitwise flag comparison
  isSSR = false
) {
  const props: Data = {}
  const attrs: Data = {}
  def(attrs, InternalObjectKey, 1)
  setFullProps(instance, rawProps, props, attrs)
  // validation
  if (__DEV__) {
    validateProps(props, instance.type)
  }

  if (isStateful) {
    // stateful
    instance.props = isSSR ? props : shallowReactive(props)
  } else {
    if (!instance.type.props) {
      // functional w/ optional props, props === attrs
      instance.props = attrs
    } else {
      // functional w/ declared props
      instance.props = props
    }
  }
  instance.attrs = attrs
}

```

凭着我的感觉应该这里会找到答案 

先看这里的 setFullProps 函数

```js
function setFullProps(
  instance: ComponentInternalInstance,
  rawProps: Data | null,
  props: Data,
  attrs: Data
) {
  const [options, needCastKeys] = normalizePropsOptions(instance.type)
  if (rawProps) {
    for (const key in rawProps) {
      const value = rawProps[key]
      // key, ref are reserved and never passed down
      if (isReservedProp(key)) {
        continue
      }
      // prop option names are camelized during normalization, so to support
      // kebab -> camel conversion here we need to camelize the key.
      let camelKey
      if (options && hasOwn(options, (camelKey = camelize(key)))) {
        props[camelKey] = value
      } else if (!isEmitListener(instance.type, key)) {
        // Any non-declared (either as a prop or an emitted event) props are put
        // into a separate `attrs` object for spreading. Make sure to preserve
        // original key casing
        attrs[key] = value
      }
    }
  }

  if (needCastKeys) {
    const rawCurrentProps = toRaw(props)
    for (let i = 0; i < needCastKeys.length; i++) {
      const key = needCastKeys[i]
      props[key] = resolvePropValue(
        options!,
        rawCurrentProps,
        key,
        rawCurrentProps[key]
      )
    }
  }
}

```

啊哦，又碰到这个函数了 normalizePropsOptions 

依然采用先不研究它的策略，先看看它返回了啥东西

哦，还是返回了一个 options ，还有一个 needCastKeys 

```js
  if (rawProps) {
    for (const key in rawProps) {
      const value = rawProps[key]
      // key, ref are reserved and never passed down
      if (isReservedProp(key)) {
        continue
      }
      // prop option names are camelized during normalization, so to support
      // kebab -> camel conversion here we need to camelize the key.
      let camelKey
      if (options && hasOwn(options, (camelKey = camelize(key)))) {
        props[camelKey] = value
      } else if (!isEmitListener(instance.type, key)) {
        // Any non-declared (either as a prop or an emitted event) props are put
        // into a separate `attrs` object for spreading. Make sure to preserve
        // original key casing
        attrs[key] = value
      }
    }
  }

```
来，分析这个逻辑

rawProps，这个就是用户传入进来的 props ，还都没有处理过

第一步就是看看 props 里面对应的 key，需不需要继续处理

这里isReservedProp 是检测 key 要不要处理

```js
export const isReservedProp = /*#__PURE__*/ makeMap(
  'key,ref,' +
    'onVnodeBeforeMount,onVnodeMounted,' +
    'onVnodeBeforeUpdate,onVnodeUpdated,' +
    'onVnodeBeforeUnmount,onVnodeUnmounted'
)
```
当key 是这些值的时候，不需要在继续处理了

```js
      let camelKey
      if (options && hasOwn(options, (camelKey = camelize(key)))) {
        props[camelKey] = value
      } else if (!isEmitListener(instance.type, key)) {
        // Any non-declared (either as a prop or an emitted event) props are put
        // into a separate `attrs` object for spreading. Make sure to preserve
        // original key casing
        attrs[key] = value
      }

```

这里的逻辑就是，先看看这个 key 属于不属于 options ，如果属于的话

就要规范化一下 key ，把烤肉串形式改成 camelize 也就是驼峰形式的命名

接着用  isEmitListener 来做了一层判断 ，如果不是 emitListener 的话，那么就保存到 attrs 里面，这时候的key 需要保存原样

那我们得去看看 isEmitListener

```js

// Check if an incoming prop key is a declared emit event listener.
// e.g. With `emits: { click: null }`, props named `onClick` and `onclick` are
// both considered matched listeners.

export function isEmitListener(comp: Component, key: string): boolean {
  let emits: ObjectEmitsOptions | undefined
  if (!isOn(key) || !(emits = normalizeEmitsOptions(comp))) {
    return false
  }
  key = key.replace(/Once$/, '')
  return (
    hasOwn(emits, key[2].toLowerCase() + key.slice(3)) ||
    hasOwn(emits, key.slice(2))
  )
}
```
这里的注释写的很清楚，就是看看 prop key 是不是在 emit event 里面声明好的

如果是的话，那么就返回 true，否则 false。

比如 emits:{click: null } 而 props 是 onCilck 或者 onclick 都可以

那么我们回到 setFullProps 来看看

如果这个东西没有在 emits 声明的话，那么就给添加到 attrs 里面了

好，但是到这里我依然没有找到答案，在说一下我的问题

在 runtime-dom 里面的 patchProps 里面，如果发现是 props 里面的 key 是 on 开头的话，那么就会给注册原生的事件，那怎么去除的？

😯，突然想到，这里不就是去除掉了嘛？  如果包含在 emits 里面的话，那么就不给添加到 attrs 里面了，但是这里是 attrs 并不是 props  。

这里需要验证一下，是先跑的 setFullProps 还是先跑的 patchProp

啊哦，我想了一下，肯定是先跑 setFullProps ，因为是必须要先初始化组件之后再处理 element 类型的组件

那好了，我去看看 patchProp 是处理的 props 还是处理的 attrs 不就好了吗

omg ，看完了，竟然不是

好吧。我有点迷了

看来只能是上 debug 来看看具体的操作了


--- 

debug 过程

先写个demo 可以调试起来

啊哦！ 终于找到答案了

其实刚刚就离答案很近了

我总结一波

---

总结

首先在组件初始化的时候会调用 initProps，而在 initProps 里面会生成 attrs 属性，在生成 attrs 属性的时候，会调用 isEmitListener 检测当前的这个 prop 是不是 emitListener

而这个 isEmitListener 就是检测这个 emitName 有没有在 emits 里面声明

如果有的话，这里的 attrs 就不会包含这个 prop 了。

并且这里还涉及到一个知识点  fallthrough ，这个的意思就是把当前组件的 attrs 属性传到子组件内

> 这个 fallthrough 下次在分析

所以只要 attrs 包含了 prop （这里可以假设是 click） ，那么就会 fallthrough 下去，如果子组件里面也有 click 事件的话，就会合并在一起。就变成了 click 一次，触发两次

因为最终这个 onClick 还会触发 patchProp 函数来添加事件监听器

其实原因都是 fallthrough 造成的。

但是重点就是在 initProps 时进行的筛选逻辑 ，也就是 isEmitListener 。

好了，至此就分析完为什么设置完 emits 之后，就不会造成触发两次 click 事件啦

> 下次需要分析 fallthrough逻辑  这个逻辑是在 renderComponentRoot 函数内做的处理 








