<template>
    <div>
    <div id="controlBar">
          <div class="toggleElement">  
            <span id="increaseStrength" @click="increaseStrength"  title="Click to increase strength">
                <font-awesome-icon :icon="['fas','redo']" transform="down-3" class="iconStyle" style="color: darkgrey"/>
            </span>
          </div>  
          <div class="toggleElement">              
            <span id="descreaseStrength" @click="decreaseStrength"  title="Click to decrease strength">
                <font-awesome-icon :icon="['fas','redo']" transform="down-3" class="iconStyle" style="color: darkgrey"/>
            </span>
          </div>  

    </div>    
    <div id="force">
        
    </div>
    </div>
</template>

<script>
/* eslint-disable */

/*
    TODO:
    1. Add json file tag attribute and convert to handle a passed in json file descriptor
    2. Itemize force control types and add control widgets
    3. Deal with the placement of controls on the svg element so that they don't change with the zoom or pan
        They need to float over the svg.
    4.     
*/
import * as d3 from "d3";
import forceData from "../../public/data/trump.json";

import { library } from "@fortawesome/fontawesome-svg-core";
import { faRedo } from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";


library.add(faRedo);

export default {
  name: "vforce",
  components: {
    faRedo,
    FontAwesomeIcon
  },
  data() {
    return {
      svgSocial: {},
      node: {},
      link: {},
      colorSocial: {},
      width: "",
      height: "",
      radius: 5,
      strength: -100,
      distance: 100,
      simulation: {},
      minZoom: 0.1,
      maxZoom: 7,
      zoom: {}
    };
  },
  mounted() {
    this.width = document.getElementById("force-panel").offsetWidth;
    this.height = document.getElementById("force-panel").offsetHeight;

    this.simulation = d3
        .forceSimulation()
        .force("link",
          d3.forceLink().id(function(d) {
            return d.id;
          })
        )
        .force("charge", 
           d3.forceManyBody()
            .strength(-100)
            .distanceMax([100])
        )
        .force("center", d3.forceCenter());

    this.zoom = d3.zoom().on("zoom", this.handleZoom);

    this.buildForce();
    this.restartForce();
  },

  methods: {

    handleZoom: function() {
      this.svgSocial.attr("transform", d3.event.transform);
      this.svgSocial.attr("scale", d3.event.scale);
    },

    dragstarted: function(d) {
      if (!d3.event.active) this.simulation.alphaTarget(0.3).restart();
      d.fx = d.x;
      d.fy = d.y;
    },

    dragged: function(d) {
      d.fx = d3.event.x;
      d.fy = d3.event.y;
    },

    dragended: function(d) {
      if (!d3.event.active) this.simulation.alphaTarget(0);
      d.fx = null;
      d.fy = null;
    },

    increaseStrength: function() {
      //this.strength *= 1.2;
      this.distance *= 1.5;
      this.simulation.force("charge", 
           d3.forceManyBody()
            .strength(this.strength)
            .distanceMax([this.distance])
        )
       this.simulation.alpha(1).restart(); 
    },

    decreaseStrength: function() {
      this.distance /= 1.5;
      this.simulation.force("charge", 
           d3.forceManyBody()
            .strength(this.strength)
            .distanceMax([this.distance])
        )
       this.simulation.alpha(1).restart(); 

    },

    buildForce() {
      let that = this;

      that.colorSocial = d3.scaleOrdinal(d3.schemeDark2);

      that.svgSocial = d3
        .selectAll("#force")
        .append("svg")
        .attr("viewBox", [
          -that.width / 2,
          -that.height / 2,
          that.width,
          that.height
        ])
        .append("g");

      d3.select("#force svg").call(that.zoom);

      that.link = that.svgSocial
        .append("g")
        .attr("class", "links")
        .selectAll("line");

      that.node = that.svgSocial
        .append("g")
        .attr("class", "nodes")
        .selectAll("circle");

        that.node.append("title").html(function(d) {
          return d.id;
        });      


      that.simulation.nodes(forceData.nodes).on("tick", that.ticked);

      that.simulation.force("link").links(forceData.links);
    },

    restartForce: function() {
      let that = this;
console.log(forceData);
      that.link = that.link.data(forceData.links);
      that.link.exit().remove();
      that.link = that.link.enter()
        .append("line")
        .attr("stroke-width", function(d) {
          return Math.sqrt(d.relation);
        });

      that.node = that.node.data(forceData.nodes)
      that.node.exit().remove();
      that.node = that.node
        .enter()
        .append("circle")
        .attr("r", that.radius)
        .attr("fill", function(d) {
          return that.colorSocial(d.detail);
        })
        .call(
          d3
            .drag()
            .on("start", that.dragstarted)
            .on("drag", that.dragged)
            .on("end", that.dragended)
        );

    },

    ticked: function() {

        this.node
          .attr("cx", function(d) {
            return d.x;
          })
          .attr("cy", function(d) {
            return d.y;
          });

        this.link
          .attr("x1", function(d) {
            return d.source.x;
          })
          .attr("y1", function(d) {
            return d.source.y;
          })
          .attr("x2", function(d) {
            return d.target.x;
          })
          .attr("y2", function(d) {
            return d.target.y;
          });
      }    
  }
};
</script>

<style>


#controlBar {
  height: 2vmax;
  background-color: lightgrey;
  border-color: darkgrey;
  border-style: solid;
  border-width: 2px;
  margin-bottom: 5px;
  font-size: 1.25vmax;
}

.toggleElement {
    margin-left: 5px;
    white-space: no-wrap;
    float: left;
    cursor: pointer;
}

.links line {
  stroke: #999;
  stroke-opacity: 0.6;
}

.nodes circle {
  stroke: #fff;
  stroke-width: 1.5px;
}
</style>
