<template>
  <movable-area class="drag-sort" :style="{height: currentList.length * height + 'px'}" id="drag">
    <movable-view
    v-for="(item, index) in currentList"
    :key="index"
    :x="0"
    :y="item.y"
    direction="vertical"
    disabled
    damping="40"
    :animation="item.animation"
    class="drag-sort-item"
    style="height:55px"
    :class="{'active': active == index, 'vh-1px-t': item.index > 0}">
      <view class="item">{{item[props.label]}}</view>
      <view class="touch-tight">
        <view class="ico_drag"></view>
      </view>
    </movable-view>
    <movable-view
    class="touch"
    :x="2000"
    @touchstart="touchstart"
    @touchmove="touchmove"
    @touchend="touchend"
    catchtouchstart
    catchtouchmove
    catchtouchend></movable-view>
  </movable-area>
</template>

<script>
export default {
  name: 'drag-sort',
  mixins: [],
  components: {},
  data () {
    return {
      height: 55, // 高度
      currentList: [],
      active: -1, // 当前激活的item
      topY: 0, // 距离顶部的距离
      deviationY: 0 // 偏移量
    }
  },
  computed: {},
  props: {
    list: {
      type: Array,
      default: () => {
        return []
      }
    },
    props: {
      type: Object,
      default: () => {
        return {
          label: 'label',
          value: 'value'
        }
      }
    }
  },
  watch: {
    list (val) {
      this.onUpdateCurrentList()
    }
  },
  created () {
    this.onUpdateCurrentList()
  },
  mounted () {
  },
  updated () {},
  filters: {},
  methods: {
    onUpdateCurrentList () {
      let arr = []
      for (const key in this.list) {
        arr.push({
          ...this.list[key],
          index: key,
          y: key * this.height,
          animation: true
        })
      }
      this.currentList = arr
    },
    touchstart (e) {
      // 计算y轴点击位置
      var query = wx.createSelectorQuery()
      query.select('#drag').boundingClientRect()
      query.exec((res) => {
        this.topY = res[0].top
        let touchY = e.mp.touches[0].clientY - res[0].top
        this.deviationY = touchY % this.height
        // console.log(touchY)
        for (const key in this.currentList) {
          if ((this.currentList[key].index * this.height < touchY) && ((this.currentList[key].index + 1) * this.height > touchY)) {
            this.active = key
            break
          }
        }
      })
    },
    touchmove (e) {
      if (this.active < 0) return
      let touchY = (e.mp.touches[0].clientY - this.topY) - this.deviationY
      // console.log(touchY)
      this.currentList[this.active].y = touchY
      this.currentList[this.active].animation = false
      for (const key in this.currentList) {
        // 跳过当前操作的item
        if (this.currentList[key].index !== this.currentList[this.active].index) {
          if (this.currentList[key].index > this.currentList[this.active].index) {
            if (touchY > this.currentList[key].index * this.height - this.height / 2) {
              this.currentList[this.active].index = this.currentList[key].index
              this.currentList[key].index = this.currentList[key].index - 1
              this.currentList[key].y = this.currentList[key].index * this.height
              break
            }
          } else {
            if (touchY < this.currentList[key].index * this.height + this.height / 2) {
              this.currentList[this.active].index = this.currentList[key].index
              this.currentList[key].index = this.currentList[key].index + 1
              this.currentList[key].y = this.currentList[key].index * this.height
              break
            }
          }
        }
      }
    },
    touchend (e) {
      if (this.active > -1) {
        this.$emit('change', {
          // 操作值
          data: (() => {
            let data = {
              ...this.currentList[this.active]
            }
            delete data.index
            delete data.y
            delete data.animation
            return data
          })(),
          // 插队的位置前面的值
          frontData: (() => {
            for (const iterator of this.currentList) {
              if (this.currentList[this.active].index - 1 === iterator.index) {
                let data = {
                  ...iterator
                }
                delete data.index
                delete data.y
                delete data.animation
                return data
              }
            }
          })()
        })
      }
      this.currentList[this.active].animation = true
      this.currentList[this.active].y = this.currentList[this.active].index * this.height
      this.active = -1
    }
  }
}
</script>

<style lang='less' scoped>
@import "~./1px.less";
.drag-sort {
  width: 100%;
}
.drag-sort-item {
  position: absolute !important;
  display: flex;
  align-items: center;
  width: 100%;
  padding: 0;
  margin: 0;
  background: #fff;
  padding: 0 15px;
  box-sizing: border-box;
  .item {
    flex: 1;
  }
  .touch-tight {
    width: 24px;
    display: flex;
    justify-content: center;
  }
}
.touch {
  height: 100%;
  width: 50px;
}
.ico_drag {
  display: inline-block;
  width: 24px;
  height: 12px;
  background: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADAAAAAYCAYAAAC8/X7cAAAAAXNSR0IArs4c6QAAAEtJREFUWAnt1cEJACAMA0B1/506moIr5FEK51+Jl0d2Vd01+JzB2X90H5jeoPwECBDIBLYlzgDj25Y4JvQAAQIERgtY4u76LHF3Aw8rGQnK3sYAXQAAAABJRU5ErkJggg==) 0 0 no-repeat;
  background-size: 100% auto;
}
.active {
  box-shadow: 0 0 40rpx #DDDDDD;
  z-index: 99;
}
</style>
