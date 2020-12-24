想要弄懂这个问题

线索是先看看如果有 .d.ts 类型声明文件

会在我们写代码的时候给与我们怎么样的类型提示

我们知道 vue3 是有内置组件提供的

所以我们先去使用官方提供的组件，看看它都是怎么给我们做类型提示的

然后，我们照葫芦画瓢，在去看看官方组件的 .d.ts 文件是什么样子的

这样我们就可以实现自己的组件类型声明文件了


![[Pasted image 20201124225023.png]]

可以看到，当我们写组件的时候，按下一个空格键 自动就会把当前组件的所有属性都显示出来了

这个就是我们写类型声明文件的目的

## 探索
那么我们需要看看官方是如何实现的

先找到 Transition 组件的源代码

在 packages/runtime-dom/components/Transition.ts  看到了以下代码
```js
export interface TransitionProps extends BaseTransitionProps<Element> {
  name?: string
  type?: typeof TRANSITION | typeof ANIMATION
  css?: boolean
  duration?: number | { enter: number; leave: number }
  // custom transition classes
  enterFromClass?: string
  enterActiveClass?: string
  enterToClass?: string
  appearFromClass?: string
  appearActiveClass?: string
  appearToClass?: string
  leaveFromClass?: string
  leaveActiveClass?: string
  leaveToClass?: string
}
```

这不就是声明了 Props 的 Interface 嘛

那我等会就照葫芦画瓢也写一个这个

先不着急，先继续往下看

别的也没啥了，先写个 button 的 .d.ts 看看能不能实现上面的效果

## 照葫芦画瓢
我先来个 button.d.ts 试试手

发现写完并没有什么卵用

那我们只好采用 googlo 大法



## 现状
好吧，研究了好半天，只有官方的可以出现类型提示，而自己写的就出不来

很奇怪

那么暂时 .d.ts 还保持原状吧

后续用 ts 重写组件的时候在看看

