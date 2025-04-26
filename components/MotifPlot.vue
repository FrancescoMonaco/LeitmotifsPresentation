<script setup>
import { onMounted, watch, ref } from 'vue'
import * as d3 from 'd3'

// Props
const props = defineProps({
  timeSeries: {
    type: Array, // Array of arrays, e.g., [[dim0_values], [dim1_values], ...]
    required: true,
  },
  dimension: {
    type: Number, // Number of dimensions
    required: true,
  },
  motifs: {
    type: Array, // Array of {start: Number, end: Number, color: String}
    default: () => [],
  },
  width: {
    type: Number,
    default: 600,
  },
  height: {
    type: Number,
    default: 200,
  },
})

const containerId = 'd3-motifplot'

// Re-render if data changes
watch(() => [props.timeSeries, props.motifs], () => {
  draw()
})

// D3 drawing logic
function draw() {
  // Clear previous svg if exists
  d3.select(`#${containerId}`).selectAll('*').remove()

  const margin = { top: 10, right: 10, bottom: 20, left: 30 }
  const svg = d3
    .select(`#${containerId}`)
    .append('svg')
    .attr('width', props.width)
    .attr('height', props.height)

  const timeSteps = props.timeSeries[0].length

  const xScale = d3.scaleLinear()
    .domain([0, timeSteps - 1])
    .range([margin.left, props.width - margin.right])

  // Calculate global min/max across all dimensions
  const flatData = props.timeSeries.flat()
  const yScale = d3.scaleLinear()
    .domain([d3.min(flatData), d3.max(flatData)])
    .range([props.height - margin.bottom, margin.top])

  const line = d3.line()
    .x((d, i) => xScale(i))
    .y(d => yScale(d))
    .curve(d3.curveMonotoneX)

  // Draw base series for each dimension
  props.timeSeries.forEach((series, dimIdx) => {
    svg.append('path')
      .datum(series)
      .attr('fill', 'none')
      .attr('stroke', d3.schemeCategory10[dimIdx % 10])
      .attr('stroke-width', 1.5)
      .attr('d', line)
  })

  // Draw motifs
  props.motifs.forEach((motif, motifIdx) => {
    props.timeSeries.forEach((series, dimIdx) => {
      const highlighted = series.map((val, i) =>
        (i >= motif.start && i <= motif.end) ? val : null
      )

      svg.append('path')
        .datum(highlighted)
        .attr('fill', 'none')
        .attr('stroke', motif.color || 'red')
        .attr('stroke-width', 3)
        .attr('d', d3.line()
          .defined(d => d !== null)
          .x((d, i) => xScale(i))
          .y(d => yScale(d))
          .curve(d3.curveMonotoneX)
        )
    })
  })
}

onMounted(() => {
  draw()
})
</script>

<template>
  <div :id="containerId" class="flex justify-center mt-4"></div>
</template>
