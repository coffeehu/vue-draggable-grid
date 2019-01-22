<template>
  <div id="app">

    <div class="display">
      <div class="display-item" v-for="item in testLayout">
        <b>{{item.i}}:</b>
        <span>[x:{{item.x}}, y:{{item.y}}, w:{{item.w}}, h:{{item.h}}]</span>
      </div>
    </div>

    <div class="tools">
      <button @click="isEdit = !isEdit">eidt</button>
      <button @click="addItem">add item</button>
      <button @click="removeItem">remove item</button>
    </div>

    <div class="map-wrapper">
      <grid-layout
        :layout="testLayout"
        :editable="isEdit">
        <div slot="base" class="map">
          <div id="map"></div>
        </div>
      </grid-layout>  
    </div>
    
  </div>
</template>

<script>
import GridLayout from './components/GridLayout'
import 'ol/ol.css';
import {Map, View} from 'ol';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';

let testLayout = [
    {"x":0,"y":0,"w":2,"h":2,"i":"0", resizable: true, draggable: true},
    {"x":1,"y":0,"w":2,"h":2,"i":"1", resizable: true, draggable: true},
    {"x":3,"y":0,"w":2,"h":1,"i":"2", resizable: true, draggable: true},
    {"x":0,"y":0,"w":2,"h":2,"i":"3", resizable: true, draggable: true},
    /*{"x":2,"y":0,"w":2,"h":4,"i":"1", resizable: null, draggable: null}*/
/*      {"x":2,"y":0,"w":2,"h":4,"i":"1", resizable: null, draggable: null},
    {"x":4,"y":0,"w":2,"h":5,"i":"2", resizable: false, draggable: false},
    {"x":6,"y":0,"w":2,"h":3,"i":"3", resizable: false, draggable: false},
    {"x":8,"y":0,"w":2,"h":3,"i":"4", resizable: false, draggable: false},
    {"x":10,"y":0,"w":2,"h":3,"i":"5", resizable: false, draggable: false},
    {"x":0,"y":5,"w":2,"h":5,"i":"6", resizable: false, draggable: false},
    {"x":2,"y":5,"w":2,"h":5,"i":"7", resizable: false, draggable: false},
    {"x":4,"y":5,"w":2,"h":5,"i":"8", resizable: false, draggable: false},
    {"x":6,"y":4,"w":2,"h":4,"i":"9", resizable: false, draggable: false},
    {"x":8,"y":4,"w":2,"h":4,"i":"10", resizable: false, draggable: false},
    {"x":10,"y":4,"w":2,"h":4,"i":"11", resizable: false, draggable: false},
    {"x":0,"y":10,"w":2,"h":5,"i":"12", resizable: false, draggable: false},
    {"x":2,"y":10,"w":2,"h":5,"i":"13", resizable: false, draggable: false},
    {"x":4,"y":8,"w":2,"h":4,"i":"14", resizable: false, draggable: false},
    {"x":6,"y":8,"w":2,"h":4,"i":"15", resizable: false, draggable: false},
    {"x":8,"y":10,"w":2,"h":5,"i":"16", resizable: false, draggable: false},
    {"x":10,"y":4,"w":2,"h":2,"i":"17", resizable: false, draggable: false},
    {"x":0,"y":9,"w":2,"h":3,"i":"18", resizable: false, draggable: false},
    {"x":2,"y":6,"w":2,"h":2,"i":"19", resizable: false, draggable: false}*/
];

export default {
  name: 'App',
  components: {
    GridLayout
  },
  mounted() {
    this.initMap();
  },
  data() {
    return {
      testLayout: testLayout,
      isEdit: true
    }
  },
  methods: {
    initMap() {
      const map = new Map({
        target: 'map',
        layers: [
          new TileLayer({
            source: new OSM()
          })
        ],
        view: new View({
          center: [0, 0],
          zoom: 0
        })
      });
    },
    addItem() {
      let item = {"x":0,"y":0,"w":2,"h":2};
      item.i = this.testLayout.length;
      this.testLayout.push(item);
    },
    removeItem() {
      this.testLayout.pop();
    }
  }
}
</script>

<style>
* {
  padding: 0;
  margin: 0;
}
.map-wrapper {
  position: fixed;
  top: 150px;
  left: 10%;
  width: 80%;
  height: 80%;
}
.map {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #e2e2e2;
}
#map {
  height: 100%;
  width: 100%;
}

.display {
  height: 80px;
  padding: 0 20px;
  background: #eee;
  color: #000;
  box-sizing: border-box;
}
.display-item {
  display: inline-block;
  margin: 10px;
}
.tools {
  height: 50px;
  line-height: 50px;
  margin-top: 10px;
  padding: 0 20px;
  background: #eee;
  color: #000;
  box-sizing: border-box;
}
.tools button {
  background: pink;
  color: #555;
  padding: 5px 10px;
  font-size: 16px;
  cursor: pointer;
}
</style>
