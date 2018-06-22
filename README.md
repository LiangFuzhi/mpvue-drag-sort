# mpvue-drag-sort

> 基于mpvue写的小程序拖拽组件

## 如何使用

```
npm i mpvue-drag-sort
```

## 基本用法
``` bash
<template>
  <dragSort :list="list" :props="props" @change="onDragSortChange"></dragSort>
</template>

<script>
import dragSort from 'dragSort'
export default {
  components: {
    dragSort
  },
  data () {
    return {
      // 对应需要显示的标题
      props: {
        label: 'label'
      },
      list: [
        {
          label: '标题1'
        },
        {
          label: '标题2'
        },
        {
          label: '标题3'
        },
        {
          label: '标题4'
        },
        {
          label: '标题5'
        }
      ]
    }
  },
  methods: {
    onDragSortChange (e) {
      console.log(e)
    }
  }
}
</script>
```
