## 基于 ResizeObserver API 动态获取内容宽度，判断是否需要el-tooltip显示溢出内容

下面封装的这个组件是基于ResizeObserver和el-tooltip(elementUI)，来判断是否需要el-tooltip显示溢出内容

```vue
// 组件封装
<template>
  <div class="overflow-tooltip">
    <span class="overflow-tooltip_wrap" ref="wrap">
      <el-tooltip
        :disabled="!showOverflowTooltip"
        :content="content"
        :placement="placement"
      >
        <template slot="content">
          <slot name="content"></slot>
        </template>
        <span>
          <slot>{{ content }}</slot>
        </span>
      </el-tooltip>
    </span>
  </div>
</template>

<script>
export default {
  name: 'OverflowTooltip',
  props: {
    content: {
      type: String,
      default: ''
    },
    placement: {
      type: String,
      default: 'bottom'
    }
  },
  data() {
    return {
      showOverflowTooltip: false,
      observer: null,
      element: null
    }
  },
  mounted() {
    this.element = this.$refs.wrap
    this.observer = new ResizeObserver(() => {
      this.updateOverflowTooltip()
    })
    this.observer.observe(this.element)
  },
  updated() {
    this.updateOverflowTooltip()
  },
  destroyed() {
    if (this.observer) {
      // this.observer.disconnect()
      this.observer.unobserve(this.element)
    }
  },
  methods: {
    updateOverflowTooltip() {
      this.$nextTick(() => {
        this.showOverflowTooltip =
          this.$refs.wrap.scrollWidth > this.$refs.wrap.clientWidth
      })
    }
  }
}
</script>

<style lang="less" scoped>
.overflow-tooltip {
  display: flex;
  flex-direction: column;
  width: 100%;
  .overflow-tooltip_wrap {
    display: inline-block;
    width: 100%;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
}
</style>
```

```vue
// 使用
<template>
    <div style="width: 150px;">
      <overflow-tooltip :content="content" :placement="'bottom'">
        <!-- template中包含的是tooltip需要显示的content -->
        <template slot="content">
          <span>template中包含的是tooltip需要显示的content</span>
        </template>
        <!-- span中的内容是页面初始化看到的content -->
        <span>span中的内容是页面初始化看到的content</span>
      </overflow-tooltip>
    </div>
</template>

<script>
import OverflowTooltip from './components/overflow-tooltip.vue'
export default {
  components: { OverflowTooltip },
  data() {
    return {
      content: '动态判断内容溢出是否显示tooltip'
    }
  }
}
</script>

```

#### 关于ResizeObserver的使用可参考如下链接：
- https://developer.mozilla.org/zh-CN/docs/Web/API/ResizeObserver
- https://zhuanlan.zhihu.com/p/41418813
