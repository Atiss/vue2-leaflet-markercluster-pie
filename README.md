# vue2-leaflet-markercluster-pie

This is a [markercluster-pie plugin](https://github.com/Leaflet/Leaflet.markercluster) extension for [vue2-leaflet package](https://github.com/KoRiGaN/Vue2Leaflet)

## Install

    // todo: npm install --save vue2-leaflet-markercluster-pie

## Demo

    git clone git@github.com:toshiroakio/vue2-leaflet-markercluster-pie.git
    cd vue2-leaflet-markercluster-pie
    yarn
    yarn example

    # or alternatively using npm
    npm install
    npm run example

Then you should be able to navigate with your browser and see the demo in http://localhost:4000/

You can see the demo code in the file [example.vue](example.vue)

## Usage

### on &lt;template&gt; add

something like this

    <v-map :zoom=10 :center="initialLocation">
      <v-icondefault :image-path="'/statics/leafletImages/'"></v-icondefault>
      <v-tilelayer url="http://{s}.tile.osm.org/{z}/{x}/{y}.png"></v-tilelayer>
      <v-marker-cluster-pie
        :key-func="keyFunc"
        :class-func="classFunc"
        :title-func="titleFunc"
        :style-func="styleFunc"
      >
      <v-marker v-for="l in locations" :key="l.id" :lat-lng="l.latlng" :icon="icon" :options="{marker_data: l}">
        <v-popup :content="l.text"></v-popup>
      </v-marker>
    </v-marker-cluster-pie>
    </v-map>

### on &lt;script&gt; add

#### option 1

In the same template file, at `<script>` part, this will make the component available only to the template in this file

    import Vue2LeafletMarkerCluster from 'vue2-leaflet-markercluster'
    ...
    export default {
      ...
      components: {
        'v-marker-cluster': Vue2LeafletMarkerCluster
        ...
      },
      ...
      methods: {
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
          const color = this.colorMap[d.data.key]; // see example
          return `
            fill: ${color};
            stroke: #444;
            background: ${color};
            border-color: #444;
          `;
        }
      },
      ...
    }

#### option 2

At main Vue configuration, this will make the component available to all templates in your app

    import Vue from 'vue'
    import Vue2LeafletMarkerClusterPie from 'vue2-leaflet-markercluster-pie'
    ...
    Vue.component('v-marker-cluster-pie', Vue2LeafletMarkerClusterPie)

### on &lt;style&gt; add

    @import "~leaflet.markercluster/dist/MarkerClusterPie.css";
    @import "~leaflet.markercluster/dist/MarkerClusterPie.Default.css";

## Access markercluster layer directly

If you need to access other markecluster methods, like [refreshClusters()](https://github.com/Leaflet/Leaflet.markercluster#refreshing-the-clusters-icon), you can do it with a ref on the markercluster vue element and using the `mapObject` property

    ...
    <v-marker-cluster-pie ref="clusterRef">
      ...
    </v-marker-cluster-pie>
    ...

    ...
    this.$refs.clusterRef.mapObject.refreshClusters()
    ...

## Develop and build

    npm install
    npm run build

## Author

[Julián Perelli](https://jperelli.com.ar/)

### Contributors

- [Ahmet Özışık](https://github.com/aozisik)
- [Nader Toukabri](https://nader.tech)

## License

MIT
