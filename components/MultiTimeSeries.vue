<script setup>
import { onMounted } from 'vue'
import * as d3 from 'd3'

onMounted(() => {
  const width = 600
  const height = 300
  const margin = { top: 20, right: 20, bottom: 30, left: 40 }

  const data = []
  const maxPoints = 200
  const nDimensions = 2

  const svg = d3
    .select('#d3-timeseries')
    .append('svg')
    .attr('width', width)
    .attr('height', height)

  const xScale = d3.scaleLinear()
    .domain([0, maxPoints])
    .range([margin.left, width - margin.right])

  const colors = ['cornflowerblue', 'crimson'] // 1st: sin, 2nd: square

  // Create a yScale for each dimension
  const yScales = [
    d3.scaleLinear().domain([-2, 2]).range([height / 2 - margin.bottom, margin.top]),  // Top half
    d3.scaleLinear().domain([-1.5, 1.5]).range([height - margin.bottom, height / 2 + margin.top]) // Bottom half
  ]

  // Add a group for each dimension
  const groups = colors.map((color, dim) => {
    return svg.append('g')
      .attr('class', `dimension-${dim}`)
  })

  // Add paths (one per dimension)
  const paths = groups.map((g, dim) =>
    g.append('path')
      .attr('fill', 'none')
      .attr('stroke', colors[dim])
      .attr('stroke-width', 2)
  )

  let x = 0

  function animate() {
    const sinVal = Math.sin(x * 0.1) + Math.random() * 0.5
    const squareVal = (Math.floor(x / 30) % 2 === 0) ? 1 + Math.cos(Math.log(x+100))*0.5 : -1 + Math.random() * 0.05
    data.push([sinVal, squareVal])

    if (data.length > maxPoints) data.shift()
    x++

    paths.forEach((path, dim) => {
      path
        .datum(data.map(d => d[dim]))
        .attr('d', d3.line()
          .x((_, i) => xScale(i))
          .y(d => yScales[dim](d))
          .curve(d3.curveMonotoneX)
        )
    })

    requestAnimationFrame(animate)
  }

  animate()
})
</script>

<template>
  <div id="d3-timeseries" class="flex justify-center mt-4"></div>
</template>
