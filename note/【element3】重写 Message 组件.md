message 是属于 service 型组件

我们组要看看这种类型的组件需要怎么来实现

最传统的套路就是暴露一个函数，然后用户调用这个函数的时候创建一个组件实例，来进行处理

那这里就会涉及到一个难点

## 难点

### 在vue3中如何创建组件实例

怎么基于一个组件选项创建对应的组件实例呢？

在 vue2 的时候我们是用 Vue.extends 来解决的，

但是到了 vue3 的时候已经没有 Vue.extends 了

### 如何管理层级问题

在 element3 中是使用的 popup-manager 来管理的，但是代码写的太乱了，我也不知道具体是怎么用的

但是设计到 message 组件的逻辑的话，它其实只涉及到一句代码

```js
 const nextZIndex = PopupManager.nextZIndex(
```

通过 popupManager 来获取一个 zindex

看了看 nextZIndex 的实现
```js

  nextZIndex: function () {
    return PopupManager.zIndex++
  },
```
其实就是在之前的 zIndex 上加一个

好，这样就理解了，这里的逻辑应该是创建一个新的 message 的时候要确保显示在最上层

关于 popupManager 别的逻辑我们先不关心，到时候在看

> 先聚焦于 Message 组件的逻辑实现


## 实现思路
- 函数逻辑处理
	- 实现一个对外的函数
	- 调用函数可传入对应的 options
	- 合并处理 options
	- 创建组件实例
		- 组件实例的 props 为处理完的 options
- 组件模板
	- html 骨架
	- close 的逻辑





## PopupManager 

看来需要研究好这个东西，所有的弹窗都会依赖这个类


## Element-plus 的实现方式
和之前的逻辑看起来没什么不同，只是小小的重构并且加了 types

不过逻辑看起来清晰了不少，可以作为主要的参考对象

