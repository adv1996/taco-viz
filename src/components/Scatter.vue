<template>
  <div>
    <svg id="scatter"/>
  </div>
</template>

<script>
/* eslint-disable */
import _ from 'lodash';
import * as d3 from 'd3';
import items from '../data/product_list.json';
export default {
  data() {
    return {
      data: [],
      height: 500,
      width: 950,
      margin: {top: 40, right: 10, bottom: 50, left: 60},
    }
  },
  mounted () {
    this.cleanData();
    this.initGraph();
  },
  methods: {
    cleanData() {
      let categories = Object.keys(items)
      this.data = []

      // collaspe and get all items and their price and calories
      // TODO exclude party packs here
      _.forEach(categories, (category) => {
        _.forEach(items[category], (item) => {
          let calories = this.getCalories(item.calories, category)
          this.data.push({
            price: item.price,
            range_calories: item.calories,
            calories: calories,
            name: item.name,
            category: category,
            ratio: parseFloat((calories / item.price).toFixed(2))
          })
        })
      })
      this.data = _.uniqBy(this.data, 'name')
    },
    getCalories(calories, category) {
      let values = calories.split('-')
      let multiplier = 1
      if (values.length > 1) {
        return Math.round((parseInt(values[1]) + parseInt(values[0])) / 2) * multiplier
      } else {
        return parseInt(values[0]) * multiplier
      }
    },
    initGraph() {
      let height = this.height
      let width = this.width
      // sets svg with appropriate height and width
      // TODO should be responsive and mobile friendly
      let svg = d3.select('#scatter')
        .attr('width', width)
        .attr('height', height)
      
      // Group inside svg for the main visualization
      const g = svg.append('g')
        .attr('transform', "translate(" + this.margin.left + "," + this.margin.top + ")")
        .attr('class', 'main_group')
      
      let maxPrice = d3.max(this.data, d => d.price)
      let minPrice = d3.min(this.data, d => d.price)
      let maxCalories = d3.max(this.data, d => d.calories)
      let minCalories = d3.min(this.data, d => d.calories)
      let xScale = d3.scaleLinear()
        .domain([0, maxCalories])
        .range([0, width - this.margin.right - this.margin.left])

      let yScale = d3.scaleLinear()
        .domain([0, maxPrice])
        .range([height - this.margin.bottom - this.margin.top, 0])
      
      let ratioExtent = d3.extent(this.data, (d) => {return d.ratio})
      let colors = ['#dadaeb','#bcbddc','#9e9ac8','#807dba','#6a51a3','#54278f','#3f007d']
      let colorScale = d3.scaleQuantize(ratioExtent, colors)
      
      let plot_radius = 3
      let g_scatter = g.append('g')
      let chosenCategory = 'ALL'
      this.data = _.orderBy(this.data, (d) => {
        if (d.category === chosenCategory) {
          return 1
        } else {
          return -1
        }
      })
      let items = g_scatter.selectAll('items')
        .data(this.data)
        .join('circle')
          .attr('class', 'items')
          .attr('r', 3)
          .attr('cx', d => xScale(d.calories))
          .attr('cy', d => yScale(d.price))
          .attr('fill', (d) => {
            if (d.category === chosenCategory || chosenCategory === 'ALL') {
              return colorScale(d.ratio)
            } else {
              return 'lightgrey'
            }
          })
          .on('mouseover', (d) => {
            console.log(d)
          })
    
      // marginal range distribution on the x-axis
      let g_barPlotX = g.append('g')
      let barPlotX = g_barPlotX.selectAll('barPlotX')
        .data(this.data)
        .join('rect')
          .attr('x', d => xScale(d.calories) - 1)
          .attr('y', height - this.margin.bottom - this.margin.top - 8)
          .attr('fill', (d) => {
            if (d.category === chosenCategory || chosenCategory === 'ALL') {
              return 'grey'
            } else {
              return 'lightgrey'
            }
          })
          .attr('width', 2)
          .attr('height', 7)
          .attr('class', 'barPlotX')
      
      // marginal range distribution on the y-axis
      let g_barPlotY = g.append('g')
      let barPlotY = g_barPlotY.selectAll('barPlotY')
        .data(this.data)
        .join('rect')
          .attr('y', d => yScale(d.price) - 1)
          .attr('x', 0)
          .attr('fill', (d) => {
            if (d.category === chosenCategory || chosenCategory === 'ALL') {
              return 'grey'
            } else {
              return 'lightgrey'
            }
          })
          .attr('width', 7)
          .attr('height', 2)
          .attr('class', 'barPlotY')
      
      // // labels for x axis
      let xLabelRange = d3.range(minCalories, maxCalories, 200)
      let g_xLabels = g.append('g')
      let xLabels = g_xLabels.selectAll('xLabels')
        .data(xLabelRange)
        .join('text')
          .attr('x', d => xScale(d) - 6)
          .attr('y', height - this.margin.bottom - this.margin.top + 15)
          .attr('dy', '0.32em')
          .text(d => d)
          .attr('class', 'xLabels')
      
      // labels for y axis
      let yLabelRange = d3.range(minPrice, maxPrice, 1)
      let g_yLabels = g.append('g')
      let yLabels = g_yLabels.selectAll('yLabels')
        .data(yLabelRange)
        .join('text')
          .attr('x', -17)
          .attr('y', d => yScale(d) + 1)
          .attr('dy', '0.32em')
          .text(d => d)
          .attr('class', 'yLabels')
      
      g_xLabels.append('line')
        .attr('x1', xScale(minCalories) - 2)
        .attr('x2', width - this.margin.left - this.margin.right + 2)
        .attr('y1', height - this.margin.bottom - this.margin.top + 5)
        .attr('y2', height - this.margin.bottom - this.margin.top + 5)
        .attr('class', 'axis')
      
      g_yLabels.append('line')
        .attr('x1', -5)
        .attr('x2', -5)
        .attr('y1', yScale(minPrice) + 1)
        .attr('y2', yScale(maxPrice) - 1)
        .attr('class', 'axis')
      
      let centerW = (width - this.margin.left - this.margin.right) / 2
      let bottomLabel = (height - this.margin.bottom)
      g.append('text')
        .attr('x', centerW)
        .attr('y', bottomLabel)
        .text('Calories')
        .attr('class', 'xaxis_label')

      let centerH = (height - this.margin.top - this.margin.bottom) / 2
      g.append('text')
        .attr('x', -centerH)
        .attr('y', -this.margin.left / 2 - 5) // why do we need to add 5 here, is there a programatic way to fix this
        .text('Price ($)')
        .attr('class', 'yaxis_label')
        .attr("transform", "rotate(-90)");

      // graph title
      g.append('text')
        .attr('x', centerW)
        .attr('y', 0)
        .text('Taco Bell Menu Items')
        .attr('class', 'title')

      let ratioOrder = _.orderBy(this.data, ['ratio'], ['asc'])
      let leastEfficient = ratioOrder[0]
      let mostEfficient = ratioOrder[this.data.length - 1]
      // color legend
      let dataLength = ratioOrder.length
      let colorLegend = [mostEfficient, ratioOrder[dataLength / 2], leastEfficient]
      let colorLegendBars = g.selectAll('colorLegendBars')
        .data(colorLegend)
        .join('rect')
          .attr('x', xScale(1250))
          .attr('y', (d, i) => {
            return yScale(4) + i * 20
          })
          .attr('height', 20)
          .attr('width', 7)
          .attr('fill', d => colorScale(d.ratio))
      
      g.append('text')
        .attr('x', xScale(1250) + 5)
        .attr('y', yScale(4) - 5)
        .text('Highest Cal / $')
        .attr('class', 'legendLabels')
      
      g.append('text')
        .attr('x', xScale(1250) + 5)
        .attr('y', yScale(4) + 70)
        .text('Least Cal / $')
        .attr('class', 'legendLabels')
    }
  },
}
</script>

<style>
.xaxis_label {
  text-anchor: middle;
  font-family: Helvetica;
  font-size: 14px;
  font-weight: 300
}
.yaxis_label {
  font-family: Helvetica;
  font-size: 14px;
  font-weight: 300
}
.title {
  text-anchor: middle;
  font-size: 18px;
  font-family: Helvetica;
  font-weight: 300;
}
.items {
  opacity: 0.8;
  stroke-width: 0.2px;
}
#scatter {
  background: left;
}
.barPlotX {
  opacity: 0.5;
}
.barPlotY {
  opacity: 0.5;
}
.xLabels {
  font-family: Helvetica;
  font-size: 10px;
  font-weight: 300
}
.yLabels {
  font-family: Helvetica;
  font-size: 10px;
  font-weight: 300
}
.axis {
  stroke: lightgray;
  stroke-opacity: 0.7;
  shape-rendering: crispEdges;
}
.anno {
  stroke: lightgray;
  stroke-opacity: 0.7;
}
.annotations {
  font-family: Helvetica;
  font-size: 9px;
  font-weight: 100
}
.legendLabels {
  font-family: Helvetica;
  font-size: 9px;
  font-weight: 100;
  text-anchor: middle;
}
</style>