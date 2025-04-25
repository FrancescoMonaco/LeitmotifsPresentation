<script setup>
import { onMounted } from 'vue'
import * as d3 from 'd3'

onMounted(() => {
  const totalPoints = 200
  const subseqLength = 30
  const data = Array.from({ length: totalPoints }, (_, i) =>
    Math.sin(i * 0.2) + Math.random() * 0.3
  )

  const width = 600
  const height = 200
  const margin = { top: 10, right: 10, bottom: 20, left: 30 }

  const svg = d3
    .select('#d3-subsequence')
    .append('svg')
    .attr('width', width)
    .attr('height', height)

  const xScale = d3.scaleLinear()
    .domain([0, totalPoints - 1])
    .range([margin.left, width - margin.right])

  const yScale = d3.scaleLinear()
    .domain([-2, 2])
    .range([height - margin.bottom, margin.top])

  const line = d3.line()
    .x((d, i) => xScale(i))
    .y((d) => yScale(d))
    .curve(d3.curveMonotoneX)

  // Draw base series
  svg.append('path')
    .datum(data)
    .attr('fill', 'none')
    .attr('stroke', '#ccc')
    .attr('stroke-width', 2)
    .attr('d', line)

  // Add subsequence path (starts empty)
  const subseqPath = svg.append('path')
    .attr('fill', 'none')
    .attr('stroke', 'mediumslateblue')
    .attr('stroke-width', 3)

  let index = 0

  function animate() {
    const end = index + subseqLength
    const subseq = data.slice(index, end)
    const padded = Array.from({ length: index }, () => null).concat(subseq)

    subseqPath.datum(padded)
      .attr('d', d3.line()
        .defined(d => d !== null)
        .x((d, i) => xScale(i))
        .y(d => yScale(d))
        .curve(d3.curveMonotoneX)
      )

    index = (index + 1) % (totalPoints - subseqLength + 1)
    setTimeout(animate, 80)
  }

  animate()
})
</script>

<template>
  <div id="d3-subsequence" class="flex justify-center mt-4"></div>
</template>
