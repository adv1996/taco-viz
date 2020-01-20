<template>
  <svg id="flow"/>
</template>

<script>
  import Recipes from '../data/taco_bell.json';
  import _ from 'lodash';
  import * as d3 from 'd3';

  export default {
    data() {
      return {
        categoryMap: [
          {
            "name": "Base",
            "children": ["Flour Torilla", "Gordita Flatbread", "Tostada Shell", "Nacho Chips", "Taco Shell", "Chalupa Shell", "Mexican Pizza Shell"]
          },
          {
            "name": "Protein",
            "children": ["Beef", "Chicken", "Eggs", "Pork"]
          },
          {
            "name": "Core",
            "children": ["Beans", "Rice", "Cheese", "Potato"]
          },
          {
            "name": "Toppings",
            "children": ["Chili", "Lettuce", "Jalapenos", "Tomatoes", "Onions", "Red Strips", "Reduced-Fat Sour Cream", "Premium Guacamole", "Purple Cabbage", "Pico de Gallo", "Fritos Chips"]
          },
          {
            "name": "Sauces",
            "children": ["Avocado Ranch Sauce", "Creamy Chipotle Sauce", "Creamy Jalepeno Sauce", "Spicy Ranch Sauce", "Sweet Chili Sauce", "Fire Roasted Salsa", "Mexican Pizza Sauce", "Nacho Cheese Sauce", "Red Sauce"]
          }
        ],
        height: 900,
        width: 1600,
        margin: {top: 50, right: 20, bottom: 20, left: 50},
      }
    },
    mounted () {
      this.initGraph();
    },
    methods: {
      initGraph() {
        const svg = d3.select('#flow')
          .attr('height', this.height + this.margin.top + this.margin.bottom)
          .attr('width', this.width + this.margin.left + this.margin.right)
        
        const g = svg.append('g')
          .attr("transform", "translate(" + this.margin.left + "," + this.margin.top + ")")

        let headers = _.map(this.categoryMap, (c) => {return c.name})
        let yscales = _.map(this.categoryMap, (c) => {
          let yscale = d3.scaleBand()
            .domain(c.children)
            .range([this.margin.top, this.height])
          return {
            "name": c.name,
            "scale": yscale,
          }
        })
        let xScale = d3.scaleBand()
          .domain(headers)
          .range([0, this.width])
        
        g.selectAll('headers')
          .data(headers)
          .enter().append('text')
          .attr('x', (d) => xScale(d))
          .attr('y', 0)
          .text((d) => d)
        
        let baseData = []
        this.categoryMap.forEach(element => {
          element.children.forEach(base => {
            baseData.push({
              "name": element.name,
              "value": base
            })
          })
        });
        g.selectAll('items')
          .data(baseData)
          .enter().append('text')
          .attr('x', (d) => xScale(d.name))
          .attr('y', (d) => {
            return _.find(yscales, { 'name': d.name }).scale(d.value)
          })
          .text((d) => d.value)


      }
    },
  }
</script>

<style lang="scss" scoped>

</style>