<template>
  <v-map :zoom=10 :center="initialLocation">
    <v-tilelayer url="http://{s}.tile.osm.org/{z}/{x}/{y}.png"></v-tilelayer>
    <!-- <v-group>
        <v-geo-json v-for="geoJson in geoJsons" :geojson="geoJson" :options="geoJsonOptions"/>
    </v-group> -->
    <v-marker-cluster-pie 
      :options="clusterOptions" 
      :key-func="keyFunc" 
      :class-func="classFunc" 
      :title-func="titleFunc" 
      :style-func="styleFunc"
      :rmax="rmax"
      @clusterclick="click()">
      <v-geo-json v-for="geoJson in geoJsons" :geojson="geoJson" :options="geoJsonOptions"/>
    </v-marker-cluster-pie>
    
  </v-map>
</template>

<script>
import L from "leaflet";
import * as Vue2Leaflet from "vue2-leaflet";
import Vue2LeafletMarkerclusterPie from "./Vue2LeafletMarkerclusterPie";

export default {
  components: {
    "v-map": Vue2Leaflet.LMap,
    "v-tilelayer": Vue2Leaflet.LTileLayer,
    "v-geo-json": Vue2Leaflet.LGeoJson,
    "v-marker-cluster-pie": Vue2LeafletMarkerclusterPie,
    "v-group": Vue2Leaflet.LFeatureGroup
  },
  methods: {
    click: function(e) {
      alert("clusterclick");
    },
    keyFunc(d) {
      //   console.log(d);
      return d.feature.properties.key;
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
    let geoJsonOptions = {
      onEachFeature: function(feature, layer) {
        layer.getLatLng = function() {
          return this.getBounds().getCenter();
        };
        layer.setLatLng = function() {};
        layer._latlng = layer.getLatLng();
      }
    };
    let geoJsons = [
      {
        type: "Feature",
        properties: {
          popupContent: "This is the Auraria West Campus",
          key: 0,
          style: {
            weight: 2,
            color: "#999",
            opacity: 1,
            fillColor: "#B0DE5C",
            fillOpacity: 0.8
          }
        },
        geometry: {
          type: "MultiPolygon",
          coordinates: [
            [
              [
                [-107.00432014465332, 39.74732195489861],
                [-107.00715255737305, 39.7462000683517],
                [-107.00921249389647, 39.74468219277038],
                [-107.01067161560059, 39.74362625960105],
                [-107.01195907592773, 39.74290029616054],
                [-107.00989913940431, 39.74078835902781],
                [-107.00758171081543, 39.74059036160317],
                [-107.00346183776855, 39.74059036160317],
                [-107.00097274780272, 39.74059036160317],
                [-107.00062942504881, 39.74072235994946],
                [-107.00020027160645, 39.74191033368865],
                [-107.00071525573731, 39.74276830198601],
                [-107.00097274780272, 39.74369225589818],
                [-107.00097274780272, 39.74461619742136],
                [-107.00123023986816, 39.74534214278395],
                [-107.00183105468751, 39.74613407445653],
                [-107.00432014465332, 39.74732195489861]
              ],
              [
                [-107.00361204147337, 39.74354376414072],
                [-107.00301122665405, 39.74278480127163],
                [-107.00221729278564, 39.74316428375108],
                [-107.00283956527711, 39.74390674342741],
                [-107.00361204147337, 39.74354376414072]
              ]
            ],
            [
              [
                [-107.00942707061768, 39.73989736613708],
                [-107.00942707061768, 39.73910536278566],
                [-107.00685214996338, 39.73923736397631],
                [-107.00384807586671, 39.73910536278566],
                [-107.00174522399902, 39.73903936209552],
                [-107.00041484832764, 39.73910536278566],
                [-107.00041484832764, 39.73979836621592],
                [-107.00535011291504, 39.73986436617916],
                [-107.00942707061768, 39.73989736613708]
              ]
            ]
          ]
        }
      },
      {
        type: "Feature",
        properties: {
          popupContent: "This is the Auraria West Campus",
          key: 1,
          style: {
            weight: 2,
            color: "#999",
            opacity: 1,
            fillColor: "#B0DE5C",
            fillOpacity: 0.8
          }
        },
        geometry: {
          type: "MultiPolygon",
          coordinates: [
            [
              [
                [-105.00432014465332, 39.74732195489861],
                [-105.00715255737305, 39.7462000683517],
                [-105.00921249389647, 39.74468219277038],
                [-105.01067161560059, 39.74362625960105],
                [-105.01195907592773, 39.74290029616054],
                [-105.00989913940431, 39.74078835902781],
                [-105.00758171081543, 39.74059036160317],
                [-105.00346183776855, 39.74059036160317],
                [-105.00097274780272, 39.74059036160317],
                [-105.00062942504881, 39.74072235994946],
                [-105.00020027160645, 39.74191033368865],
                [-105.00071525573731, 39.74276830198601],
                [-105.00097274780272, 39.74369225589818],
                [-105.00097274780272, 39.74461619742136],
                [-105.00123023986816, 39.74534214278395],
                [-105.00183105468751, 39.74613407445653],
                [-105.00432014465332, 39.74732195489861]
              ],
              [
                [-105.00361204147337, 39.74354376414072],
                [-105.00301122665405, 39.74278480127163],
                [-105.00221729278564, 39.74316428375108],
                [-105.00283956527711, 39.74390674342741],
                [-105.00361204147337, 39.74354376414072]
              ]
            ],
            [
              [
                [-105.00942707061768, 39.73989736613708],
                [-105.00942707061768, 39.73910536278566],
                [-105.00685214996338, 39.73923736397631],
                [-105.00384807586671, 39.73910536278566],
                [-105.00174522399902, 39.73903936209552],
                [-105.00041484832764, 39.73910536278566],
                [-105.00041484832764, 39.73979836621592],
                [-105.00535011291504, 39.73986436617916],
                [-105.00942707061768, 39.73989736613708]
              ]
            ]
          ]
        }
      }
    ];
    return {
      rmax: 35,
      colorMap: {
        "0": "red",
        "1": "#00FF00"
      },
      geoJsons,
      geoJsonOptions,
      clusterOptions: {},
      initialLocation: L.latLng(39.73989736613708, -107.00942707061768)
    };
  },
  mounted() {
    setTimeout(() => {
      console.log("done");
      this.$nextTick(() => {
        this.clusterOptions = { disableClusteringAtZoom: 11 };
      });
    }, 5000);
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