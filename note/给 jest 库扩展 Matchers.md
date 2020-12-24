在做 element3 的时候有个需求是需要用第三方库来扩展 jest 的 matchers

这里是通过 jest-vtu 来进行扩展

那怎么在一个第三方库扩展 jest 的 .d.ts 呢？

## 步骤

### 创建声明文件
因为是扩展全局属性，所以要用下面的 declare global 的写法来处理

// jest-vtu/src/index.d.ts
```js
export {};

declare global {
  namespace jest {
    interface Matchers<R, T = {}> {
      /**
       * @description
       * Check whether the given wrapper has certain classes within its `class` attribute.
       *
       * You must provide at least one class
       * @example
       *
       * Button.vue
       * <template>
       *   <button class="el-button--small">
       *   </button>
       * </template>
       *
       * const wrapper = mount(Button)
       *
       * expect(wrapper).toHaveClass(`el-button--small`)
       * expect(deleteButton).not.toHaveClass(`el-button--foo`)
       */
      toHaveClass(className: string): R;
    }
  }
}

```

### 设置 tsconfig.js
因为 jest-vtu 是属于第三方库（也就是只存在于 node_modules 里面）

想让 jest-vtu 里面的类型声明文件生效的话，还需要在你的项目内配置 tsconfig.json

```json
// tsconfig.json
{
	  "types": ["jest", "jest-vtu"]
}
```

必须加上 jest-vtu


这样的话才可以生效
> 前提是你的项目是一个 ts 项目