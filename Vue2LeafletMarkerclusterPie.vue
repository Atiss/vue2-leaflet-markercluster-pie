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
const rmax = 35; // максимальный радиус кластера

function serializeXmlNode(xmlNode) {
  if (typeof window.XMLSerializer != "undefined") {
    return new window.XMLSerializer().serializeToString(xmlNode);
  } else if (typeof xmlNode.xml != "undefined") {
    return xmlNode.xml;
  }
  return "";
}

function defineClusterIconProto(
  cluster,
  keyFunc,
  classFunc,
  titleFunc,
  labelClass,
  icon_class
) {
  function bakeThePie(options) {
    /*data and valueFunc are required*/
    if (!options.data || !options.valueFunc) {
      return "";
    }
    let data = options.data,
      valueFunc = options.valueFunc,
      r = options.outerRadius ? options.outerRadius : 28, //Default outer radius = 28px
      rInner = options.innerRadius ? options.innerRadius : r - 10, //Default inner radius = r-10
      strokeWidth = options.strokeWidth ? options.strokeWidth : 1, //Default stroke is 1
      pathClassFunc = options.pathClassFunc
        ? options.pathClassFunc
        : function() {
            return "";
          }, //Class for each path
      pathTitleFunc = options.pathTitleFunc
        ? options.pathTitleFunc
        : function() {
            return "";
          }, //Title for each path
      pieClass = options.pieClass ? options.pieClass : "marker-cluster-pie", //Class for the whole pie
      pieLabel = options.pieLabel ? options.pieLabel : d3.sum(data, valueFunc), //Label for the whole pie
      pieLabelClass = options.pieLabelClass
        ? options.pieLabelClass
        : "marker-cluster-pie-label", //Class for the pie label
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
    let vis = d3
      .select(svg)
      .data([data])
      .attr("class", pieClass)
      .attr("width", w)
      .attr("height", h);

    let arcs = vis
      .selectAll("g.arc")
      .data(donut.value(valueFunc))
      .enter()
      .append("svg:g")
      .attr("class", "arc")
      .attr("transform", "translate(" + origo + "," + origo + ")");

    arcs
      .append("svg:path")
      .attr("class", pathClassFunc)
      .attr("stroke-width", strokeWidth)
      .attr("d", arc)
      .append("svg:title")
      .text(pathTitleFunc);

    vis
      .append("text")
      .attr("x", origo)
      .attr("y", origo)
      .attr("class", pieLabelClass)
      .attr("text-anchor", "middle")
      //.attr('dominant-baseline', 'central')
      /*IE doesn't seem to support dominant-baseline, but setting dy to .3em does the trick*/
      .attr("dy", ".3em")
      .text(pieLabel);
    //Return the svg-markup rather than the actual element
    return serializeXmlNode(svg);
  } // для кластера
  let children = cluster.getAllChildMarkers(),
    n = children.length, //Get number of markers in cluster
    strokeWidth = 1, //Set clusterpie stroke width
    r = rmax - 2 * strokeWidth - (n < 10 ? 12 : n < 100 ? 8 : n < 1000 ? 4 : 0), //Calculate clusterpie radius...
    iconDim = (r + strokeWidth) * 2, //...and divIcon dimensions (leaflet really want to know the size)
    data = d3
      .nest() //Build a dataset for the pie chart
      .key(keyFunc)
      .entries(children, d3.map),
    //bake some svg markup
    html = bakeThePie({
      data: data,
      valueFunc: function(d) {
        return d.values.length;
      },
      strokeWidth: 1,
      outerRadius: r,
      innerRadius: r - 10,
      pieClass: "cluster-pie",
      pieLabel: n,
      pieLabelClass: labelClass,
      pathClassFunc: classFunc,
      pathTitleFunc: titleFunc
    }),
    //Create a new divIcon and assign the svg markup to the html property
    myIcon = new L.DivIcon({
      html: html,
      className: icon_class,
      iconSize: new L.Point(iconDim, iconDim)
    });
  return myIcon;
} // для создания иконки кластера

const props = {
  keyFunc: {
    type: Function,
    default: d => 0
  },
  classFunc: {
    type: Function,
    default: d => ""
  },
  titleFunc: {
    type: Function,
    default: d => ""
  },
  options: {
    type: Object,
    default() {
      return {
        maxClusterRadius: 2 * rmax,
        removeOutsideVisibleBounds: false,
        singleMarkerMode: false
        // animate: false
        // chunkedLoading: true,
        // spiderfyOnMaxZoom: false,
        // disableClusteringAtZoom: 17
        // spiderfyOnMaxZoom: false,
        // showCoverageOnHover: false,
        // zoomToBoundsOnClick: false
      };
    }
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
    this.mapObject = L.markerClusterGroup(
      Object.assign({}, this.options, {
        iconCreateFunction: this.getClusterIcon()
      })
    );
    L.DomEvent.on(this.mapObject, this.$listeners);
    propsBinder(this, this.mapObject, props);
    this.ready = true;
    this.parentContainer = findRealParent(this.$parent);
    this.parentContainer.addLayer(this);
  },
  beforeDestroy() {
    this.parentContainer.removeLayer(this);
  },
  methods: {
    getClusterIcon() {
      const context = this;
      return function(cluster) {
        return defineClusterIconProto(
          cluster,
          context.keyFunc,
          context.classFunc,
          context.titleFunc,
          "marker-cluster-pie-label",
          "marker-cluster"
        );
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
    }
  }
};
</script>
<style>
.marker-cluster-pie g.arc {
  fill-opacity: 0.5;
}

.marker-cluster-pie-label {
  font-size: 14px;
  font-weight: bold;
  font-family: sans-serif;
}

.marker-cluster {
  border-radius: 35px !important;
  background-color: #fff;
}

.marker-cluster-call-pie-label {
  font-size: 14px;
  font-weight: bold;
  font-family: serif;
}

.marker-cluster-call {
  border-radius: 35px !important;
  background-color: #efefef;
}
</style>
