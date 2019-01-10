<template>
  <v-map :zoom=10 :center="initialLocation">
    <v-icondefault></v-icondefault>
    <v-tilelayer url="http://{s}.tile.osm.org/{z}/{x}/{y}.png"></v-tilelayer>
    <v-marker-cluster-pie 
      :options="clusterOptions" 
      :key-func="keyFunc" 
      :class-func="classFunc" 
      :title-func="titleFunc" 
      :style-func="styleFunc"
      :rmax="rmax"
      @clusterclick="click()">
      <!-- require add data to marker -->
      <v-marker v-for="l in locations" :key="l.id" :lat-lng="l.latlng" :icon="icon" :options="{marker_data: l}">
        <v-popup :content="l.text"></v-popup>
      </v-marker>
    </v-marker-cluster-pie>
  </v-map>
</template>

<script>
// import L from "leaflet";
import * as Vue2Leaflet from "vue2-leaflet";
import Vue2LeafletMarkerclusterPie from "./Vue2LeafletMarkerclusterPie";
import iconUrl from "leaflet/dist/images/marker-icon.png";
import shadowUrl from "leaflet/dist/images/marker-shadow.png";

function rand(n) {
  let max = n + 0.1;
  let min = n - 0.1;
  return Math.random() * (max - min) + min;
}

export default {
  components: {
    "v-map": Vue2Leaflet.LMap,
    "v-tilelayer": Vue2Leaflet.LTileLayer,
    "v-icondefault": Vue2Leaflet.LIconDefault,
    "v-marker": Vue2Leaflet.LMarker,
    "v-popup": Vue2Leaflet.LPopup,
    "v-marker-cluster-pie": Vue2LeafletMarkerclusterPie
  },
  methods: {
    click: function(e) {
      alert("clusterclick");
    },
    keyFunc(d) {
      return d.options.marker_data[this.iconStyleField];
    },
    classFunc(d) {
      return "cluster-marker-segment-color_" + d.data.key;
    },
    titleFunc(d) {
      return `count: ${d.data.values.length}`;
    },
    styleFunc(d) {
      const color = this.colorMap[d.data.key];
      return `
        fill: ${color};
        stroke: #444;
        background: ${color};
        border-color: #444;
      `;
    }
  },
  data() {
    let locations = [];
    for (let i = 0; i < 100; i++) {
      locations.push({
        id: i,
        style_field: (Math.random() * 10) >> 0,
        latlng: Vue2Leaflet.L.latLng(rand(-34.9205), rand(-57.953646)),
        text: "Hola " + i
      });
    }
    let icon = Vue2Leaflet.L.icon(
      Object.assign({}, Vue2Leaflet.L.Icon.Default.prototype.options, {
        iconUrl,
        shadowUrl
      })
    );
    return {
      rmax: 35,
      iconStyleField: "style_field",
      locations,
      icon,
      clusterOptions: {},
      initialLocation: Vue2Leaflet.L.latLng(-34.9205, -57.953646),
      colorMap: {
        "0": "black",
        "1": "#F00000",
        "2": "#FF0000",
        "3": "#FFF000",
        "4": "#FFFF00",
        "5": "#FFFFF0",
        "6": "#FFFFFF",
        "7": "#0FFFFF",
        "8": "#00FFFF",
        "9": "#000FFF",
        "10": "#0000FF"
      }
    };
  }
};
</script>

<style>
@import "~leaflet/dist/leaflet.css";
@import "~leaflet.markercluster/dist/MarkerCluster.css";
@import "~leaflet.markercluster/dist/MarkerCluster.Default.css";
html,
body {
  height: 100%;
  margin: 0;
}
</style>