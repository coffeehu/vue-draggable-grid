<template>
  <div class="grid-background">
    <div 
      v-for="item in items"
      :style="style"
      class="grid-background-item"></div>
  </div>
</template>

<script>
export default {
  name: 'GridBackground',
  props: {
    layout: {
      type: Array
    },
    colWidth: {
      type: Number
    },
    colHeight: {
      type: Number
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
  },
  mounted() {
    
  },
  watch: {
    colWidth(val) {
      this.createStyle();
    },
    colHeight(val) {
      this.createStyle();
    }
  },
  computed: {
    items() {
      return new Array(this.colNum * this.rowNum);
    }
  },
  data () {
    return {
      innerColWidth: 0,
      innerColHeight: 0,
      style: {}
    }
  },
  methods: {
    createStyle() {
      this.style = {
        height: this.colHeight + 'px',
        width: this.colWidth + 'px'
      }
    }
  }
}
</script>

<style scoped>
.grid-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  border-left: 1px solid pink;
  border-top: 1px solid pink;
}
.grid-background-item {
  float: left;
  border-right: 1px solid pink;
  border-bottom: 1px solid pink;
  box-sizing: border-box;
}
</style>
