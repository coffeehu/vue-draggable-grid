<template>
  <div 
    class="grid-item"
    :class="{'grid-item-dragging': isDragging, 'grid-item-resizing': isResizing}"
    :style="style">
    <div
      class="grid-item-content"
      @mousedown="dragStart"
      @mouseup="dragEnd">
      item-{{i}}
    </div>

    <span 
      class="resizable-handle" 
      @mousedown="resizeStart"
      @mouseup="resizeEnd"></span>
  </div>

</template>

<script>
import {getControlPosition, createCoreData} from './util.js';

export default {
  name: 'GridItem',
  inject: ['eventBus'],
  props: {
    i: {
      require: true
    },
    w: {
      type: Number
    },
    h: {
      type: Number
    },
    x: {
      type: Number
    },
    y: {
      type: Number
    },
    colWidth: {
      type: Number
    },
    colHeight: {
      type: Number
    },
    colNum: {
      type: Number
    },
    rowNum: {
      type: Number
    }
  },
  created() {
    this.innerX = this.x;
    this.innerY = this.y;
    this.innerW = this.w;
    this.innerH = this.h;
    this.innerWidth = this.colWidth;
    this.innerHeight = this.colHeight;
  },
  data () {
    return {
      style: {},
      innerWidth: NaN, // this.colWidth
      innerHeight: NaN, // this.colHeight
      //--------drag------
      isDragging: false,
      dragPosition: null, // 记录了坐标
      lastX: NaN,
      lastY: NaN,
      innerX: NaN, // this.x
      innerY: NaN, // this.y
      //--------resize------
      isResizing: false,
      resizeInfo: null, //记录了高宽
      lastResizeX: NaN,
      lastResizeY: NaN,
      innerW: NaN, // this.w
      innerH: NaN // this.h
    }
  },
  watch: {
    colWidth(val) {
      this.left = (this.colWidth * this.x);
      this.innerWidth = this.colWidth;
      this.createStyle();
    },
    colHeight(val) {
      this.top = (this.colHeight * this.y);
      this.innerHeight = this.colHeight;
      this.createStyle();
    },
    x() {
      this.createStyle()
    },
    y() {
      this.createStyle()
    },
    w() {
      this.createStyle()
    },
    h() {
      this.createStyle()
    }
  },
  methods: {
    createStyle() {
      let style = {
        width: (this.innerW * this.colWidth) + 'px',
        height: (this.innerH * this.colHeight) + 'px',
        left: (this.x * this.colWidth) + 'px',
        top: (this.y * this.colHeight) + 'px'
      }

      if (this.isDragging) {
        style.top = this.dragPosition.top + 'px';
        style.left = this.dragPosition.left + 'px';
      }

      if(this.isResizing) {
        style.width = this.resizeInfo.width + 'px';
        style.height = this.resizeInfo.height + 'px';
      }

      this.style = style;
    },
    dragStart(event) {
      this.isDragging = true;
      /*
        getControlPosition()
        返回 {x:Number, y:Number}
        是鼠标与 parentOffset 的偏移量

        这里目的是记录当前的坐标
      */
      const position = getControlPosition(event);
      this.lastX = position.x;
      this.lastY = position.y;

      let newPosition = {top: 0, left: 0};
      let parentRect = event.target.offsetParent.offsetParent.getBoundingClientRect();
      let clientRect = event.target.getBoundingClientRect();
      newPosition.left = clientRect.left - parentRect.left;
      newPosition.top = clientRect.top - parentRect.top;
      this.dragPosition = newPosition;

      document.onmousemove = (event) => {
        const position = getControlPosition(event);
        if (!position) return;
        /*
        createCoreData()
        返回的对象：{deltaX:Number, deltaY:Number, ...}
        代表当前坐标与last坐标的偏移量 
        */
        const coreEvent = createCoreData(this.lastX, this.lastY, position.x, position.y);
        let newPosition = {top: 0, left: 0};
        newPosition.left = this.dragPosition.left + coreEvent.deltaX;
        newPosition.top = this.dragPosition.top + coreEvent.deltaY;
        this.dragPosition = newPosition;
        this.createStyle();
        this.lastX = position.x;
        this.lastY = position.y;
      }
    },
    dragEnd(evt) {
      if(this.isDragging) this.dragEndHandler();
      this.eventBus.$emit('compact');
    },
    dragEndHandler() {
      this.isDragging = false;
      document.onmousemove = null;
      this.dragPosition = {top: 0, left: 0};

      let parentRect = event.target.offsetParent.offsetParent.getBoundingClientRect();
      let clientRect = event.target.getBoundingClientRect();
      let newPosition = {top: 0, left: 0};
      newPosition.left = clientRect.left - parentRect.left;
      newPosition.top = clientRect.top - parentRect.top;
      let {x, y} = this.calcXY(newPosition.top, newPosition.left);
      this.innerX = x;
      this.$emit('update:x', x);
      this.innerY = y;
      this.$emit('update:y', y);
      this.createStyle();
    },
    resizeStart(event) {
      this.isResizing = true;
      const position = getControlPosition(event);
      this.lastResizeX = position.x;
      this.lastResizeY = position.y;

      let newSize = {width: 0, height: 0};
      newSize.width = this.colWidth * this.innerW;
      newSize.height = this.colHeight * this.innerH;
      this.resizeInfo = newSize;

      document.onmousemove = (event) => {
        const position = getControlPosition(event);
        if (!position) { //拖拽时，当鼠标移出 Layout 的范围时，返回的 postion 为 undefined
          this.resizeEndHandle();
          return;
        }

        const coreEvent = createCoreData(this.lastResizeX, this.lastResizeY, position.x, position.y);
        let newSize = {width: 0, height: 0};
        newSize.width = this.resizeInfo.width + coreEvent.deltaX;
        newSize.height = this.resizeInfo.height + coreEvent.deltaY;
        this.resizeInfo = newSize;
        this.lastResizeX = position.x;
        this.lastResizeY = position.y;
        //-----移动的时候，检测碰撞
        let {w, h} = this.calcWH(this.resizeInfo.width, this.resizeInfo.height);
        this.$emit('update:w', w);
        this.$emit('update:h', h);
        //-----
        this.createStyle();
        this.eventBus.$emit('compact');
      }
    },
    resizeEnd(event) {
      this.resizeEndHandle();
    },
    resizeEndHandle() {
      if(!this.isResizing) return;
      this.isResizing = false;
      document.onmousemove = null;
      let {w, h} = this.calcWH(this.resizeInfo.width, this.resizeInfo.height);
      this.innerW = w;
      this.innerH = h;
      this.$emit('update:w', w);
      this.$emit('update:h', h);
      this.createStyle();
      this.eventBus.$emit('compact');
    },
    calcXY(top, left) { //计算 {x, y} 坐标
      let x = Math.round(left / this.colWidth);
      let y = Math.round(top / this.colHeight);
      x = x < 0 ? 0 : Math.abs(x);
      y = y < 0 ? 0 : Math.abs(y);
      x = x > (this.colNum-this.innerW) ? (this.colNum-this.innerW) : x;
      y = y > (this.rowNum-this.innerH) ? (this.rowNum-this.innerH) : y;
      return {x, y};
    },
    calcWH(width, height) { // 计算当前高宽属于的 innerW，innerH 的值
      let w = Math.round(width / this.colWidth);
      let h = Math.round(height / this.colHeight);
      w = w < 1 ? 1 : w;
      h = h < 1 ? 1 : h;
      return {w, h};
    }
  }
}
</script>

<style scoped>
.grid-item {
  position: absolute;
  background: burlywood;
  border: 1px solid #555;
  text-align: center;
  box-sizing: border-box;
  cursor: move;
  transition: all .2s ease-out;

  -webkit-touch-callout: none; /* iOS Safari */
  -webkit-user-select: none; /* Chrome/Safari/Opera */
  -khtml-user-select: none; /* Konqueror */
  -moz-user-select: none; /* Firefox */
  -ms-user-select: none; /* Internet Explorer/Edge */
  user-select: none; /* Non-prefixed version, currently*/
}
.grid-item-content {
  width: 100%;
  height: 100%;
}
.grid-item > .resizable-handle {
    position: absolute;
    width: 20px;
    height: 20px;
    bottom: 0;
    right: 0;
    background: url('data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBzdGFuZGFsb25lPSJubyI/Pg08IS0tIEdlbmVyYXRvcjogQWRvYmUgRmlyZXdvcmtzIENTNiwgRXhwb3J0IFNWRyBFeHRlbnNpb24gYnkgQWFyb24gQmVhbGwgKGh0dHA6Ly9maXJld29ya3MuYWJlYWxsLmNvbSkgLiBWZXJzaW9uOiAwLjYuMSAgLS0+DTwhRE9DVFlQRSBzdmcgUFVCTElDICItLy9XM0MvL0RURCBTVkcgMS4xLy9FTiIgImh0dHA6Ly93d3cudzMub3JnL0dyYXBoaWNzL1NWRy8xLjEvRFREL3N2ZzExLmR0ZCI+DTxzdmcgaWQ9IlVudGl0bGVkLVBhZ2UlMjAxIiB2aWV3Qm94PSIwIDAgNiA2IiBzdHlsZT0iYmFja2dyb3VuZC1jb2xvcjojZmZmZmZmMDAiIHZlcnNpb249IjEuMSINCXhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiIHhtbDpzcGFjZT0icHJlc2VydmUiDQl4PSIwcHgiIHk9IjBweCIgd2lkdGg9IjZweCIgaGVpZ2h0PSI2cHgiDT4NCTxnIG9wYWNpdHk9IjAuMzAyIj4NCQk8cGF0aCBkPSJNIDYgNiBMIDAgNiBMIDAgNC4yIEwgNCA0LjIgTCA0LjIgNC4yIEwgNC4yIDAgTCA2IDAgTCA2IDYgTCA2IDYgWiIgZmlsbD0iIzAwMDAwMCIvPg0JPC9nPg08L3N2Zz4=');
    background-position: bottom right;
    padding: 0 3px 3px 0;
    background-repeat: no-repeat;
    background-origin: content-box;
    box-sizing: border-box;
    cursor: se-resize;
}
.grid-item-dragging {
  opacity: 0.8;
  z-index: 3;
  transition: none;
}
.grid-item-resizing {
  transition: none;
}
</style>
