<template>
  <svg id="bubble"/>
</template>

<script>
  import Ingredients from '../data/ingredients.json';
  import _ from 'lodash';
  import * as d3 from 'd3';
  export default {
    mounted () {
      this.initGraph();
    },
    methods: {
      initGraph() {
        let diameter = 1000;
        var color = ['#fee391','#fec44f','#fe9929','#ec7014','#cc4c02','#993404','#662506'];
        let bubble = d3.pack()
          .size([diameter, diameter])
          .padding(1.5);

        const svg = d3.select('#bubble')
          .attr('height', 1000)
          .attr('width', 1000)
        
        let nodes = d3.hierarchy(Ingredients)
            .sum((d) => { return d.size; });
        
        let node = svg.selectAll('.node')
          .data(bubble(nodes).descendants())
          .enter()
          .filter((d) => {
              return  !d.children
          })
          .append('g')
            .attr("class", 'node')
            .attr('transform', function(d) {
                return "translate(" + d.x + "," + d.y + ")";
            });

        node.append("circle")
          .attr("r", function(d) {
              return d.r;
          })
          .style("fill", function(d,i) {
              return color[i % 7];
          });

        node.append("text")
          .attr("dy", ".2em")
          .style("text-anchor", "middle")
          .text(function(d) {
              return d.data.name.substring(0, d.r / 3);
          })
          .attr("font-family", "sans-serif")
          .attr("font-size", function(d){
              return d.r/5;
          })
          .attr("fill", "white");

        node.append("text")
          .attr("dy", "1.3em")
          .style("text-anchor", "middle")
          .text(function(d) {
              return d.data.size;
          })
          .attr("font-family",  "Gill Sans", "Gill Sans MT")
          .attr("font-size", function(d){
              return d.r/5;
          })
          .attr("fill", "white");

        d3.select(self.frameElement)
          .style("height", 1000 + "px");
      }
    },
  }
</script>

<style lang="scss" scoped>

</style>