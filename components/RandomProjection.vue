<script setup>
import { ref, watch, onMounted } from 'vue'
import * as d3 from 'd3'

const numPoints = 200
const numProjections = ref(2)
const binWidth = ref(0.5)

const width = 600
const height = 200
const margin = { top: 20, right: 20, bottom: 20, left: 20 }

const points = ref(generatePoints())
const projections = ref(generateProjections(numProjections.value))

let svg, pointElements, vectorElements, tickElements

onMounted(() => {
  svg = d3.select('#projection-svg')
    .append('svg')
    .attr('width', width)
    .attr('height', height)

  pointElements = svg.selectAll('circle')
    .data(points.value)
    .enter()
    .append('circle')
    .attr('r', 4)
    .attr('cx', d => xScale(d.x))
    .attr('cy', d => yScale(d.y))
    .attr('fill', 'gray')

  vectorElements = svg.append('g')
  tickElements = svg.append('g')

  updateVisualization()
})

const xScale = d3.scaleLinear()
  .domain([-1.2, 1.2])
  .range([margin.left, width - margin.right])

const yScale = d3.scaleLinear()
  .domain([-1.2, 1.2])
  .range([height - margin.bottom, margin.top])

function generatePoints() {
  return Array.from({ length: numPoints }, () => ({
    x: Math.random() * 2 - 1,
    y: Math.random() * 2 - 1,
  }))
}

function generateProjections(n) {
  return Array.from({ length: n }, () => {
    const angle = Math.random() * 2 * Math.PI
    return [Math.cos(angle), Math.sin(angle)]
  })
}

function regeneratePoints() {
  points.value = generatePoints()
  projections.value = generateProjections(numProjections.value)

  pointElements.data(points.value)
    .join(
      enter => enter.append('circle')
        .attr('r', 4)
        .attr('cx', d => xScale(d.x))
        .attr('cy', d => yScale(d.y))
        .attr('fill', 'gray'),
      update => update
        .transition()
        .duration(300)
        .attr('cx', d => xScale(d.x))
        .attr('cy', d => yScale(d.y)),
      exit => exit.remove()
    )

  updateVisualization()
}

function updateVisualization() {
  const bins = points.value.map(p => {
    return projections.value.map(([vx, vy]) => {
      const dot = p.x * vx + p.y * vy
      return Math.floor(dot / binWidth.value)
    }).join('-')
  })

  const uniqueBins = Array.from(new Set(bins))
  const colorScale = d3.scaleOrdinal()
    .domain(uniqueBins)
    .range(d3.schemeCategory10.concat(d3.schemePastel1).concat(d3.schemeSet3))

  pointElements
    .data(points.value)
    .transition()
    .duration(500)
    .attr('fill', (_, i) => colorScale(bins[i]))

  // Draw vectors
  vectorElements.selectAll('line')
    .data(projections.value)
    .join('line')
    .attr('x1', xScale(0))
    .attr('y1', yScale(0))
    .attr('x2', d => xScale(d[0]))
    .attr('y2', d => yScale(d[1]))
    .attr('stroke', 'white')
    .attr('stroke-width', 2)
    .attr('marker-end', 'url(#arrowhead)')

  // Draw bin cuts
  const ticks = []
  projections.value.forEach(([vx, vy]) => {
    const maxDot = Math.sqrt(2)  // Max projection length for [-1,1]x[-1,1]
    for (let k = -Math.ceil(maxDot / binWidth.value); k <= Math.ceil(maxDot / binWidth.value); k++) {
      const d = k * binWidth.value
      ticks.push({
        x: vx * d,
        y: vy * d,
        vx,
        vy,
      })
    }
  })

  tickElements.selectAll('line')
    .data(ticks)
    .join('line')
    .attr('x1', d => xScale(d.x - d.vy * 0.12))
    .attr('y1', d => yScale(d.y + d.vx * 0.12))
    .attr('x2', d => xScale(d.x + d.vy * 0.12))
    .attr('y2', d => yScale(d.y - d.vx * 0.12))
    .attr('stroke', 'yellow')
    .attr('stroke-width', 5)

  // Define arrowheads
  svg.select('defs')?.remove()
  svg.append('defs').append('marker')
    .attr('id', 'arrowhead')
    .attr('viewBox', '0 -5 10 10')
    .attr('refX', 5)
    .attr('refY', 0)
    .attr('markerWidth', 6)
    .attr('markerHeight', 6)
    .attr('orient', 'auto')
    .append('path')
    .attr('d', 'M0,-5L10,0L0,5')
    .attr('fill', 'white')
}

watch(binWidth, updateVisualization)
watch(numProjections, (newVal) => {
  projections.value = generateProjections(newVal)
  updateVisualization()
})
</script>

<template>
  <div class="flex flex-col items-center space-y-4">
    <div class="flex space-x-4">
      <div>
        <label>Number of projections: {{ numProjections }}</label>
        <input type="range" min="1" max="10" v-model="numProjections" />
      </div>
      <div>
        <label>Bin width: {{ binWidth }}</label>
        <input type="range" min="0.1" max="1" step="0.05" v-model="binWidth" />
      </div>
      <button @click="regeneratePoints" class="bg-blue-500 hover:bg-blue-600 text-white px-3 py-1 rounded">
        Regenerate Points
      </button>
    </div>
    <div id="projection-svg"></div>
  </div>
</template>
