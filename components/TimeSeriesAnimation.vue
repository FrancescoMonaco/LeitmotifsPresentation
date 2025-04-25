<script setup>
import { onMounted } from 'vue'
import * as d3 from 'd3'

onMounted(() => {
  const width = 600
  const height = 200
  const margin = { top: 10, right: 10, bottom: 20, left: 30 }

  const data = []
  const maxPoints = 200

  const svg = d3
    .select('#d3-timeseries')
    .append('svg')
    .attr('width', width)
    .attr('height', height)

  const xScale = d3.scaleLinear().domain([0, maxPoints]).range([margin.left, width - margin.right])
  const yScale = d3.scaleLinear().domain([-2, 2]).range([height - margin.bottom, margin.top])

  const line = d3
    .line()
    .x((d, i) => xScale(i))
    .y((d) => yScale(d))
    .curve(d3.curveMonotoneX)

  const path = svg
    .append('path')
    .attr('fill', 'none')
    .attr('stroke', 'cornflowerblue')
    .attr('stroke-width', 2)

  let x = 0

  function animate() {
    const nextY = Math.sin(x * 0.1) + Math.random() * 0.5
    data.push(nextY)
    if (data.length > maxPoints) data.shift()
    x++

    path
      .datum(data)
      .attr('d', line)

    requestAnimationFrame(animate)
  }

  animate()
})
</script>

<template>
  <div id="d3-timeseries" class="flex justify-center mt-4"></div>
</template>

