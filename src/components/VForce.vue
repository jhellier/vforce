<template>
    <div id="force">
        
    </div>
</template>

<script>
/* eslint-disable */

import * as d3 from "d3";
import forceData from "../../public/data/trump.json";

export default {
  name: "vforce",
  data() {

      

      return {
          svgSocial: {},
          width: '',
          height: '',
          radius: 5,
          simulation: d3.forceSimulation()
            .force("link", d3.forceLink().id(function(d) { return d.id; }))
            .force("charge", d3.forceManyBody().strength(-10).distanceMax([100]))
            .force("center", d3.forceCenter()),
          hello: 'what',
          minZoom: 0.1,
          maxZoom: 7,
          zoom: d3.zoom().on('zoom', this.handleZoom)  
      }
  },
  mounted() {

      this.width = document.getElementById("force-panel").offsetWidth;
      this.height = document.getElementById("force-panel").offsetHeight;

    //   this.simulation = d3.forceSimulation()
    //     .force("link", d3.forceLink().id(function(d) { return d.id; }))
    //     .force("charge", d3.forceManyBody())
    //     .force("center", d3.forceCenter());

    this.buildForce();
    
  },

  methods: {

    handleZoom: function() {
        this.svgSocial.attr("transform", d3.event.transform);
        this.svgSocial.attr('scale', d3.event.scale);
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




    buildForce() {
      let that = this;


    var colorSocial = d3.scaleOrdinal(d3.schemeDark2);  


      that.svgSocial = d3
        .selectAll("#force")
        .append("svg")
            .attr('viewBox',[-that.width/2, -that.height/2, that.width, that.height])
        .append("g")
        

    d3.select('svg').call(that.zoom);

    var link = that.svgSocial.append("g")
      .attr("class", "links")
    .selectAll("line")
    .data(forceData.links)
    .enter().append("line")
      .attr("stroke-width", function(d) { return Math.sqrt(d.relation); })

  var node = that.svgSocial.append("g")
      .attr("class", "nodes")
    .selectAll("circle")
    .data(forceData.nodes)
    .enter().append("circle")
      .attr("r", that.radius)
      .attr("fill", function(d) { return colorSocial(d.detail); })
      .call(d3.drag()
          .on("start", that.dragstarted)
          .on("drag", that.dragged)
          .on("end", that.dragended));

  node.append("title")
      .html(function(d) { 
          return d.id;
        });

  that.simulation
      .nodes(forceData.nodes)
      .on("tick", ticked);

  that.simulation.force("link")
      .links(forceData.links);

  function ticked() {

    // node.attr("cx", function(d) { return d.x = Math.max(that.radius, Math.min(that.width - that.radius, d.x)); })
    //     .attr("cy", function(d) { return d.y = Math.max(that.radius, Math.min(that.height - that.radius, d.y)); });        

    node
        .attr("cx", function(d) { return d.x; })
        .attr("cy", function(d) { return d.y; });

        link
        .attr("x1", function(d) { return d.source.x; })
        .attr("y1", function(d) { return d.source.y; })
        .attr("x2", function(d) { return d.target.x; })
        .attr("y2", function(d) { return d.target.y; });

  }

    }
  }
};
</script>

<style>

.links line {
  stroke: #999;
  stroke-opacity: 0.6; }

.nodes circle {
  stroke: #fff;
  stroke-width: 1.5px; }

</style>
