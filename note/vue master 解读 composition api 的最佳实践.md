- ref vs reactive
	- 使用 ref

- compoistiion api dealing with arguments(接受的参数)
	- 提前抛出错误 vs 兼容（wrap）
		- 判断是不是个 ref

- return computed > return ref
	- 用 computed 包裹下，可以防止被用户修改
	- 相当于内部的这个值就变成私有的了 
	- 并且 computed 本身就是不允许改变
	- 目的就是只读不可写
	![[Pasted image 20210123194813.png]]
	![[Pasted image 20210123195041.png]]
	![[Pasted image 20210123195444.png]]

- watch vs 生命周期
	- watch	
		- el 元素 change 后还可以生效
	- 生命周期就不可以了
	
	![[Pasted image 20210123192802.png]]
	
- composables 
	- 所有的 use 放置文件夹
	
	
	
## code 

展示了 reactive vs ref 的两种使用方式

```
<template>
  <table>
    <thead>
      <tr>
        <template v-for="(column, index) in tableHeads" :key="index">
          <th>{{ column.label }}</th>
        </template>
      </tr>
    </thead>
    <tbody>
      <template v-for="(row, index) in rows" :key="index">
        <tr>
          <td v-for="(val, key) in row" :key="key">{{ val }}</td>
        </tr>
      </template>
    </tbody>
    <slot></slot>
  </table>
</template>

<script>
import {
  onMounted,
  computed,
  defineComponent,
  reactive,
  unref,
  ref,
  provide,
  getCurrentInstance,
  watchEffect,
  isReactive,
  toRefs,
  Ref,
  ComponentInternalInstance
} from 'vue'
export default defineComponent({
  name: 'ElNewTable',
  props: ['data'],
  // setup(props) {
  //   // 1. 通过 state(reactive) 的方式来书写逻辑
  //   // 优点是 利用的 reactive 不需要写 .value (不用满天飞)
  //   // 缺点是 不方便封装

  //   // 给所有的子组件提供 table 实例
  //   const instance = getCurrentInstance()
  //   provide('table', instance.proxy)

  //   const state = reactive({
  //     columns: [],
  //     rows: [],
  //     tableHeads: []
  //   })

  //   function registryColumn(column) {
  //     state.columns.push(column)
  //   }

  //   // 计算出 tableHeads
  //   watchEffect(() => {
  //     state.tableHeads = state.columns.map((columnVM) => {
  //       const { prop, label } = columnVM.props
  //       return {
  //         prop,
  //         label
  //       }
  //     })
  //   })

  //   // 计算 rows
  //   watchEffect(() => {
  //     state.rows = props.data.map((item) => {
  //       return state.tableHeads.reduce((result, { prop: key }) => {
  //         result[key] = item[key]
  //         return result
  //       }, {})
  //     })
  //   })

  //   return {
  //     ...toRefs(state),
  //     registryColumn
  //   }
  // }
  setup(props) {
    // 1. 全部使用 ref 来构建逻辑
    // 优点： 可以做到完全逻辑独立
    // 缺点： 需要调用 .value 不过 eslint 会给出提示
    const { data } = toRefs(props)

    useProvide()

    // 实际执行的主流程
    const { columns, registryColumn } = useColumns()
    const { tableHeads } = useTableHeads(columns)
    const { rows } = useRows(data, tableHeads)

    return {
      registryColumn,
      tableHeads,
      rows
    }
  }
})

function useProvide() {
  const instance = getCurrentInstance()
  // 给所有的子组件提供 table 实例
  provide('table', instance.proxy)
}

function useColumns() {
  const columns = ref([])
  function registryColumn(column) {
    columns.value.push(column)
  }

  return {
    columns,
    registryColumn
  }
}

// 实现 tableHeaders
function useTableHeads(columns) {
  const tableHeads = computed(() => {
    return columns.value.map((columnVM) => {
      const { prop, label } = columnVM.props
      return {
        prop,
        label
      }
    })
  })

  return {
    tableHeads
  }
}

// 实现 rows
function useRows(data, tableHeads) {
  const rows = computed(() => {
    return data.value.map((item) => {
      return tableHeads.value.reduce((result, { prop: key }) => {
        result[key] = item[key]
        return result
      }, {})
    })
  })

  return {
    rows
  }
}
</script>

<style></style>

```