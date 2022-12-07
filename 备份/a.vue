<template>
  <div class="row" ref="panel">
    <!-- <draggable class="list-group" :list="layout" group="people" @change="log"> -->
    <div
      class="list-group-item"
      v-for="element in layout"
      :key="element.name"
      :ref="`widget${element.name}`"
      draggable
      @dragstart="handleDragStart(element)"
      @dragover.prevent="handleDragOver($event)"
      @dragenter="handleDragEnter($event, element)"
      @dragend="handleDragEnd"
    >
      <!-- {{ element.name }} {{ index }} -->
      <x-bar :name="element.name" :ele="element" :h="element.h" />
    </div>
    <!-- </draggable> -->
  </div>
</template>
<script>
// import draggable from "vuedraggable";
import Bar from "@/components/Bar";
import cloneDeep from "lodash/cloneDeep";
export default {
  name: "app",
  display: "App Page",
  components: {
    // draggable,
    [Bar.name]: Bar,
  },
  data() {
    return {
      layout: [
        { name: "John", id: 1, h: 200 },
        { name: "Joao", id: 2, h: 340 },
        { name: "Jean", id: 3, h: 230 },
        { name: "Gerard", id: 4, h: 276 },
        { name: "Juan", id: 5, h: 333 },
        { name: "Edgard", id: 6, h: 444 },
        { name: "Johnson", id: 7, h: 210 },
      ],
      ending: null,
      dragging: null,
    };
  },
  mounted() {
    this.init();
    this.addResizeEvent();
  },
  methods: {
    init() {
      const { width: wrapperWidth } = this.$refs.panel.getBoundingClientRect();
      const itemWidth = Math.ceil((wrapperWidth - 24 * 3) / 2);
      const posY = new Array(2).fill(24);
      const elements = cloneDeep(this.layout);
      for (let i = 0; i < elements.length; i++) {
        const idx = i % 2;
        const widget = this.$refs[`widget${elements[i].name}`];
        const { height } = widget[0].getBoundingClientRect();
        widget[0].style.width = itemWidth + "px";
        Object.assign(elements[i], {
          x: 24 * (idx + 1) + idx * itemWidth,
          y: posY[idx],
          h: height,
        });
        widget[0].style.transform = `translate(${
          24 * (idx + 1) + idx * itemWidth
        }px, ${posY[idx]}px)`;
        posY[idx] += height + 24;
      }
      this.layout = elements;
    },
    addResizeEvent() {
      window.addEventListener("resize", () => {
        console.log("trigger");
        this.init();
      });
    },
    log: function (evt) {
      window.console.log(evt);
    },
    handleDragStart(item) {
      this.dragging = item;
    },
    swapXY(srcItem, dstItem) {
      const { x, y } = srcItem;
      Object.assign(srcItem, { x: dstItem.x, y: dstItem.y });
      Object.assign(dstItem, { x, y });
    },
    evalDragItemStyle(srcItem, dstItem) {
      const { x, y, name } = srcItem;
      const diff = dstItem.h - srcItem.h;
      console.log(diff, "---diff");
      // 同列情况
      if (x === dstItem.x) {
        // 找出交换后，位置在下方的模块
        let A = y > dstItem.y ? srcItem : dstItem;
        console.log(y > dstItem.y);
        let B = y < dstItem.y ? srcItem : dstItem;
        let resizeY = y > dstItem.y ? A.y + diff : A.y - diff;
        this.$refs[
          `widget${A.name}`
        ][0].style.transform = `translate(${A.x}px, ${resizeY}px)`;
        A.y = resizeY;
        this.$refs[
          `widget${B.name}`
        ][0].style.transform = `translate(${B.x}px, ${B.y}px)`;
      } else {
        this.$refs[
          `widget${name}`
        ][0].style.transform = `translate(${x}px, ${y}px)`;
        this.$refs[
          `widget${dstItem.name}`
        ][0].style.transform = `translate(${dstItem.x}px, ${dstItem.y}px)`;
        for (let i = 0; i < this.layout.length; i++) {
          const { x, y, name: eleName } = this.layout[i];
          if (eleName !== srcItem.name && x === srcItem.x && y > srcItem.y) {
            this.$refs[
              `widget${eleName}`
            ][0].style.transform = `translate(${x}px, ${y - diff}px)`;
            this.layout[i].y = y - diff;
          }
          if (eleName !== dstItem.name && x === dstItem.x && y > dstItem.y) {
            this.$refs[
              `widget${eleName}`
            ][0].style.transform = `translate(${x}px, ${y + diff}px)`;
            this.layout[i].y = y + diff;
          }
        }
      }
    },
    handleDragEnd() {
      if (this.ending.id === this.dragging.id) {
        return;
      }
      let newItems = [...this.layout];
      const src = newItems.indexOf(this.dragging);
      const dst = newItems.indexOf(this.ending);
      this.swapXY(this.dragging, this.ending);
      newItems.splice(src, 1, ...newItems.splice(dst, 1, newItems[src]));
      this.layout = newItems;
      this.$nextTick(() => {
        this.evalDragItemStyle(this.dragging, this.ending);
        this.dragging = null;
        this.ending = null;
      });
    },
    handleDragOver(e) {
      // 首先把div变成可以放置的元素，即重写dragenter/dragover
      // 在dragenter中针对放置目标来设置
      e.dataTransfer.dropEffect = "move";
    },
    handleDragEnter(e, item) {
      // 为需要移动的元素设置dragstart事件
      e.dataTransfer.effectAllowed = "move";
      this.ending = item;
    },
  },
};
</script>
<style lang="less">
.row {
  position: relative;
}
.list-group-item {
  // border-bottom: 1px solid #ccc;
  position: absolute;
  display: inline-block;
}
</style>