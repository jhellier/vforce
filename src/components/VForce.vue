<template>
    <div>
    <div id="controlBar">
          <div class="toggleElement">  
            <span id="increaseStrength" @click="increaseStrength"  title="Click to increase strength">
                <font-awesome-icon :icon="['fas','arrow-up']" transform="down-3" class="iconStyle" style="color: #5f5c5c"/>
            </span>
          </div>  
          <div class="toggleElement">              
            <span id="descreaseStrength" @click="decreaseStrength"  title="Click to decrease strength">
                <font-awesome-icon :icon="['fas','arrow-down']" transform="down-3" class="iconStyle" style="color: #5f5c5c"/>
            </span>
          </div>  
          <div class="toggleElement">              
            <span id="addNode" @click="addNode"  title="Click to add node">
                <font-awesome-icon :icon="['fas','plus']" transform="down-3" class="iconStyle" style="color: #5f5c5c"/>
            </span>
          </div>  
          <div class="toggleElement">              
            <span id="zoomIn" @click="zoomIn"  title="Click to zoom in">
                <font-awesome-icon :icon="['fas','expand']" transform="down-3" class="iconStyle" style="color: #5f5c5c"/>
            </span>
          </div>  
          <div class="toggleElement">              
            <span id="zoomOut" @click="zoomOut"  title="Click to zoom out">
                <font-awesome-icon :icon="['fas','compress']" transform="down-3" class="iconStyle" style="color: #5f5c5c"/>
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
    4.  Added branch to bug234   
*/
import * as d3 from "d3";
import forceData from "../../public/data/trump.json";

import { library } from "@fortawesome/fontawesome-svg-core";
import { faCompress } from "@fortawesome/free-solid-svg-icons";
import { faArrowUp } from "@fortawesome/free-solid-svg-icons";
import { faArrowDown } from "@fortawesome/free-solid-svg-icons";
import { faExpand} from "@fortawesome/free-solid-svg-icons";
import { faPlus } from "@fortawesome/free-solid-svg-icons";

import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";


library.add(faCompress);
library.add(faArrowUp);
library.add(faArrowDown);
library.add(faExpand);
library.add(faPlus);

export default {
  name: "vforce",
  components: {
    faCompress,
    faArrowUp,
    faArrowDown,
    faExpand,
    faPlus,
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
      minZoom: 0.2,
      maxZoom: 1,
      currentZoomLevel: 0,
      zoom: {}
    };
  },
  beforeCreate: function() {
    console.log('Before Created');
    

  },
  created: function() {
    console.log('Created');
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

  },
  mounted: function() {
    console.log('Mounted');
    this.width = document.getElementById("force-panel").offsetWidth;
    this.height = document.getElementById("force-panel").offsetHeight;

    this.buildForce();
    this.restartForce();
  },
  updated: function() {
    console.log('Updated');
  },
  destroyed: function() {
    console.log('Destroyed');
  },

  methods: {

    handleZoom: function() {
      console.log(d3.event.transform);
      this.svgSocial.attr("transform", d3.event.transform);
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

    zoomIn: function() {
      // Set this so that the zoom is smooth
      // One way is to loop with small increment and delay each step
      
      let that = this;
      let timer = setInterval(zoomIt, 15);
      let zoomFactorEnd = that.minZoom;
      let zoomFactorStep = 0.05;
      let scaleFactor = that.maxZoom;
      function zoomIt() {
        scaleFactor = scaleFactor - zoomFactorStep;
        if (scaleFactor < zoomFactorEnd) {
          that.currentZoomLevel = scaleFactor;
          clearInterval(timer);
          return;
        }
        that.svgSocial.attr('transform', 'scale(' + scaleFactor + ')');
      }
    },

    zoomOut: function() {
      // Set this so that the zoom is smooth
      // One way is to loop with small increment and delay each step
      let that = this;
      let timer = setInterval(zoomIt, 15);
      let zoomFactorEnd = that.maxZoom;
      let zoomFactorStep = 0.05;
      let scaleFactor = that.minZoom;

      function zoomIt() {

        if (that.svgSocial.attr('transform')) {
          let tempFactor = that.svgSocial.attr('transform').match(/([0-9]+)/g);
          tempFactor = tempFactor[tempFactor.length - 1];
          if (tempFactor == zoomFactorEnd) {
            clearInterval(timer);
            return;
          }
        } 
          scaleFactor = scaleFactor + zoomFactorStep;
        
        if (scaleFactor > zoomFactorEnd) {
          that.currentZoomLevel = scaleFactor;
          clearInterval(timer);
          return;
        }
        that.svgSocial.attr('transform', 'scale(' + scaleFactor + ')');
      }
    },

    addNode: function() {
     
      forceData.nodes.push({"id":"JOE BLOW"});
      forceData.links.push({"source":"JACK WEISSELBERG",
                             "target":"JOE BLOW",
                             "relation": "Parent/child",
                              "detail": "https://www.nytimes.com/2016/05/24/business/dealbook/donald-trump-relationship-bankers.html?_r=0"
                              });
      this.restartForce();                             
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
    },

    restartForce: function() {
      let that = this;
      that.node = that.node.data(forceData.nodes, function(d) { return d.id; })
      that.node.exit().remove();
      that.node = that.node
        .enter()
        .append("circle")
        .attr("r", that.radius)
        .attr("fill", function(d) {
          return '#303030';
        })
        .style('stroke','darkblue')
        .style('stroke-width',1)
        .on('mouseover', function(d) {
          d3.select(this).attr('fill','red')
        })
        .on('mouseout', function(d) {
          d3.select(this).attr('fill','#303030')
        })
        .call(
          d3
            .drag()
            .on("start", that.dragstarted)
            .on("drag", that.dragged)
            .on("end", that.dragended)
        ).merge(that.node)


    that.link = that.link.data(forceData.links,  function(d) { return d.source.id + "-" + d.target.id; });
      that.link.exit().remove();
      that.link = that.link.enter()
        .append("line")
        .style('stroke','darkblue')
        .style('stroke-width',1)        
        .merge(that.link);


        that.node.selectAll('title').remove();
        that.node.append("title").html(function(d) {
          return d.id;
        });

      that.simulation.nodes(forceData.nodes).on("tick", that.ticked);

      that.simulation.force("link").links(forceData.links);
        this.simulation.alpha(1).restart();

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
