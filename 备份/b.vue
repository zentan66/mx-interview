<template>
  <grid-layout
    :layout.sync="layout"
    :row-height="300"
    :is-draggable="true"
    :is-resizable="true"
    :is-mirrored="false"
    :breakpoints="{ lg: 1920, md: 1440, sm: 768, xs: 414, xxs: 0 }"
    :cols="{ lg: 7, md: 5, sm: 4, xs: 2, xxs: 1 }"
    :vertical-compact="true"
    :margin="[10, 10]"
    :use-css-transforms="true"
    auto-size
    :max-rows="7"
    responsive
    @breakpoint-changed="breakpointChangedEvent"
  >
    <grid-item
      v-for="item in layout.filter((m) => m.visible)"
      :x="item.x"
      :y="item.y"
      :w="item.w"
      :h="item.h"
      :i="item.i"
      :max-w="4"
      :max-h="3"
      :key="item.i"
    >
      <div class="box">
        <p>{{ item.i }}</p>
      </div>
    </grid-item>
  </grid-layout>
</template>
<script>
import VueGridLayout from "vue-grid-layout";
import Bar from "@/components/Bar";
import cloneDeep from "lodash/cloneDeep";

const SIZE_MAP = {};

export default {
  name: "app",
  display: "App Page",
  components: {
    [Bar.name]: Bar,
    GridLayout: VueGridLayout.GridLayout,
    GridItem: VueGridLayout.GridItem,
  },
  data() {
    return {
      componentName: "x-bar",
      layout: [
        { x: 0, y: 0, w: 2, h: 1, i: "0", visible: true },
        { x: 1, y: 0, w: 1, h: 1, i: "1", visible: true },
        { x: 2, y: 5, w: 1, h: 3, i: "6", visible: true },
        { x: 0, y: 5, w: 1, h: 1, i: "7", visible: true },
        { x: 1, y: 5, w: 1, h: 3, i: "8", visible: true },
        { x: 2, y: 3, w: 1, h: 3, i: "9", visible: true },
        { x: 3, y: 4, w: 1, h: 1, i: "10", visible: true },
        { x: 4, y: 4, w: 1, h: 1, i: "11", visible: true },
        { x: 5, y: 10, w: 1, h: 1, i: "12", visible: true },
      ],
      cacheHeight: {},
      ending: null,
      dragging: null,
    };
  },
  methods: {
    breakpointChangedEvent: function (newBreakpoint, newLayout) {
      let cloneLayout = cloneDeep(newLayout);
      cloneLayout = cloneLayout.filter((m) => !m.i.startsWith("temp_"));
      const beyondHeightItems = cloneLayout.filter((m) => m.h > 1);
      beyondHeightItems.forEach((item) => {
        item.h = 2;
      });
      this.$nextTick(() => {
        if (this.layout.length === cloneLayout.length) {
          cloneLayout.push({
            x: 6,
            y: 6,
            w: 1,
            h: 1,
            i: `temp_${Math.random().toString(36).slice(2)}`,
            visible: false,
          });
        }
        this.cacheHeight = cloneLayout.reduce((prev, cur) => {
          prev[cur.i] = `${cur.h}_${cur.w}`;
          return prev;
        }, {});
        this.layout = cloneLayout;
      });
    },
  },
};
</script>
<style lang="less">
.box {
  height: 100%;
  width: 100%;
  border: 1px solid #ddd;
  background-color: lightblue;
  font-size: 30px;
}
</style>
