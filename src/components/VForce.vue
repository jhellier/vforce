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
      let sWidth = document.body.clientWidth;
      let sHeight = document.body.clientHeight;
      
      let sim = d3.forceSimulation()
        .force("link", d3.forceLink().id(function(d) { return d.id; }))
        .force("charge", d3.forceManyBody())
        .force("center", d3.forceCenter(sWidth / 2, sHeight / 2));

      return {
          svgSocial: {},
          simulation: sim
      }
  },
  mounted() {
    this.buildForce();
  },

  methods: {


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
            .attr("viewBox", "0 0 1300 1075")
            //.style('fill','blue')
        .append("g")
            //.attr("transform", "translate(150,150)");

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
      .attr("r", 5)
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
    link
        .attr("x1", function(d) { return d.source.x; })
        .attr("y1", function(d) { return d.source.y; })
        .attr("x2", function(d) { return d.target.x; })
        .attr("y2", function(d) { return d.target.y; });

    node
        .attr("cx", function(d) { return d.x; })
        .attr("cy", function(d) { return d.y; });
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
