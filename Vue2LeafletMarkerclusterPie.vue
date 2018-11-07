<template>
  <div style="display: none;">
    <slot v-if="ready"></slot>
  </div>
</template>

<script>
import L from "leaflet";
import "leaflet.markercluster";

import { findRealParent, propsBinder } from "vue2-leaflet";

const d3 = require("d3");
// const rmax = 100; // max cluster radius

function serializeXmlNode(xmlNode) {
  if (typeof window.XMLSerializer != "undefined") {
    return new window.XMLSerializer().serializeToString(xmlNode);
  } else if (typeof xmlNode.xml != "undefined") {
    return xmlNode.xml;
  }
  return "";
}

const props = {
  keyFunc: {
    type: Function,
    custom: true,
    default: d => 0
  },
  classFunc: {
    type: Function,
    custom: true,
    default: d => ""
  },
  titleFunc: {
    type: Function,
    custom: true,
    default: d => ""
  },
  styleFunc: {
    type: Function,
    custom: true,
    default: d => ""
  },
  rmax: {
    type: Number,
    custom: true,
    default: 35
  },
  options: {
    type: Object,
    default: d => ({})
  }
};

export default {
  props,
  data() {
    return {
      ready: false
    };
  },
  mounted() {
    this.parentContainer = findRealParent(this.$parent);
    this.mapObject = L.markerClusterGroup(
      Object.assign({}, this.options, {
        iconCreateFunction: this.getClusterIcon()
      })
    );
    L.DomEvent.on(this.mapObject, this.$listeners);
    propsBinder(this, this.mapObject, props);
    this.ready = true;
    this.parentContainer.addLayer(this);
  },
  beforeDestroy() {
    this.parentContainer.removeLayer(this);
  },
  methods: {
    bakeThePie(options) {
      /*data and valueFunc are required*/
      if (!options.data || !options.valueFunc) {
        return "";
      }
      let data = options.data,
        valueFunc = options.valueFunc,
        r = options.outerRadius ? options.outerRadius : 28, //Default outer radius = 28px
        rInner = options.innerRadius ? options.innerRadius : (r / 1.7) >> 0, //Default inner radius = (r / 1.7) >> 0
        strokeWidth = options.strokeWidth ? options.strokeWidth : 1, //Default stroke is 1
        origo = r + strokeWidth, //Center coordinate
        w = origo * 2, //width and height of the svg element
        h = w,
        donut = d3.pie(),
        arc = d3
          .arc()
          .innerRadius(rInner)
          .outerRadius(r);
      // donut = d3.layout.pie(),
      // arc = d3.svg.arc().innerRadius(rInner).outerRadius(r);

      //Create an svg element
      let svg = document.createElementNS(d3.namespaces.svg, "svg");
      //Create the pie chart
      // console.log(data);
      let vis = d3
        .select(svg)
        .data([data])
        .attr("class", options.pieClass)
        .attr(
          "style",
          `background-color: #FFF; border-radius: ${this.rmax}px !important;`
        )
        .attr("width", w)
        .attr("height", h);

      // const segmentColor =

      let arcs = vis
        .selectAll("g.arc")
        .data(donut.value(valueFunc))
        .enter()
        .append("svg:g")
        .attr("class", "arc")
        .attr("transform", "translate(" + origo + "," + origo + ")");

      arcs
        .append("svg:path")
        .attr("class", this.classFunc)
        .attr("style", this.styleFunc)
        .attr("stroke-width", strokeWidth)
        .attr("d", arc)
        .append("svg:title")
        .text(this.titleFunc);

      vis
        .append("text")
        .attr("x", origo)
        .attr("y", origo)
        .attr("class", options.pieLabelClass)
        .attr(
          "style",
          `
            font-size: ${(this.rmax / 2.5) >> 0}px;
            font-weight: bold; 
            font-family: sans-serif;
          `
        )
        .attr("text-anchor", "middle")
        //.attr('dominant-baseline', 'central')
        /*IE doesn't seem to support dominant-baseline, but setting dy to .3em does the trick*/
        .attr("dy", ".3em")
        .text(options.pieLabel);
      //Return the svg-markup rather than the actual element
      return serializeXmlNode(svg);
    },
    generateHtmlIcon(html, icon_class, iconDim) {
      return new L.DivIcon({
        html: html,
        className: icon_class,
        iconSize: new L.Point(iconDim, iconDim)
      });
    },
    getClusterIcon() {
      const self = this;
      return function(cluster) {
        const children = cluster.getAllChildMarkers(),
          n = children.length, //Get number of markers in cluster
          strokeWidth = 1, //Set clusterpie stroke width
          r =
            self.rmax -
            2 * strokeWidth -
            (n < 10 ? 12 : n < 100 ? 8 : n < 1000 ? 4 : 0), //Calculate clusterpie radius...
          iconDim = (r + strokeWidth) * 2, //...and divIcon dimensions (leaflet really want to know the size)
          data = d3
            .nest() //Build a dataset for the pie chart
            .key(self.keyFunc)
            .entries(children, d3.map),
          //bake some svg markup
          html = self.bakeThePie({
            data: data,
            valueFunc: d => d.values.length,
            strokeWidth: 1,
            outerRadius: r,
            innerRadius: (r / 1.7) >> 0,
            pieClass: "marker-cluster-pie",
            pieLabel: n,
            pieLabelClass: "marker-cluster-pie-label"
          });
        return self.generateHtmlIcon(html, "marker-cluster-pie-icon", iconDim);
      };
    },
    addLayer(layer, alreadyAdded) {
      if (!alreadyAdded) {
        this.mapObject.addLayer(layer.mapObject);
      }
    },
    removeLayer(layer, alreadyRemoved) {
      if (!alreadyRemoved) {
        this.mapObject.removeLayer(layer.mapObject);
      }
    },
    setRmax(v, o) {
      const err = new Error(
        `rmax = ${v}. rmax not be set more than once. The window will be reloaded.`
      );
      console.error(err);
      window.confirm(err);
      window.location.reload();
      // if (v !== o && !isNaN(v)) {
      //   // todo: redraw marker cluster
      // }
    }
  }
};
</script>
<style>
</style>
