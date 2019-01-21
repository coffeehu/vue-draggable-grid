<template>
  <div class="grid-layout" ref="contain">
    <slot name="base"></slot>

    <grid-background 
      :layout="layout"
      :col-width="colWidth"
      :col-height="colHeight"
      :col-num="colNum"
      :row-num="rowNum"></grid-background>

    <grid-item 
      v-for="item in originalLayout"
      :w.sync="item.w"
      :h.sync="item.h"
      :x.sync="item.x"
      :y.sync="item.y"
      :i="item.i"
      :key="item.i"
      :col-width="colWidth"
      :col-height="colHeight"
      :col-num="colNum"
      :row-num="rowNum"></grid-item>

  </div>
</template>

<script>
import Vue from 'vue';
import GridBackground from './GridBackground';
import GridItem from './GridItem.vue';

let eventBus = new Vue();

export default {
  name: 'GridLayout',
  provide() {
      return {
          eventBus: eventBus
      }
  },
  components: {
    GridBackground,
    GridItem
  },
  props: {
    layout: {
      type: Array
    },
    colNum: {
      type: Number,
      default: 12
    },
    rowNum: {
      type: Number,
      default: 12
    }
  },
  created() {
    this.compact();
    this.originalLayout = this.layout;
    eventBus.$on('compact', this.compact);
  },
  mounted() {
    this.containWidth = this.$refs.contain.offsetWidth;
    this.containHeight = this.$refs.contain.offsetHeight;
  },
  data() {
    return {
      originalLayout: [],
      containWidth: 0,
      containHeight: 0
    }
  },
  computed: {
    colWidth() {
      return this.containWidth / this.colNum;
    },
    colHeight() {
      return this.containHeight / this.rowNum;
    },
    sortedLayout() { //将 layout 排序
      return this.layout.sort(function(a, b) {
        if (a.y > b.y || (a.y === b.y && a.x > b.x)) {
          return 1;
        }
        return -1;
      });
    }
  },
  methods: {
    compact() { //碰撞检测
      let compared = [];
      for(let i=0, l=this.sortedLayout.length; i<l; i++) {
        let item = this.sortedLayout[i];
        this.compactItem(compared, item);
        compared.push(item);
      }
    },
    compactItem(compared, item) {
      for(let i=0, l=compared.length; i<l; i++) {
        if( this.isCollide(compared[i], item) ) {
          item.y = compared[i].h + compared[i].y; //碰撞的话，就 y 轴下移
          break;
        }
      }
    },
    isCollide(l1, l2) { //两个方块是否碰撞
      if(l1 === l2) return false;
      if (l1.x + l1.w <= l2.x) return false;
      if (l1.x >= l2.x + l2.w) return false;
      if (l1.y + l1.h <= l2.y) return false;
      if (l1.y >= l2.y + l2.h) return false;
      return true;
    },

  }
}
</script>

<style scoped>
.grid-layout {
  position: relative;
  width: 100%;
  height: 100%;
}
</style>
