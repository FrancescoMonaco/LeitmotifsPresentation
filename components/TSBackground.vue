<script setup>
import { ref, onBeforeUnmount } from 'vue' // Import ref
import { onSlideEnter, onSlideLeave } from '@slidev/client'
import * as d3 from 'd3' // Use a namespace import for clarity

// Create a ref to hold the DOM element
const animatedBgEl = ref(null)

let intervalId
let svg // Keep track of the SVG element associated with this instance

const createSvg = () => {
  // Ensure the element is available before proceeding
  if (!animatedBgEl.value) {
    console.error('Target element for animation not found.')
    return;
  }

  // Select the container using the template ref
  const container = d3.select(animatedBgEl.value)
  container.selectAll('svg').remove() // Ensure no duplicates *within this specific container*

  svg = container
    .append('svg')
    .attr('width', window.innerWidth)
    .attr('height', window.innerHeight)
    .style('position', 'absolute')
    .style('top', 0)
    .style('left', 0)
    .style('z-index', -1) // Using -1 is often safer for backgrounds than 0
    .style('pointer-events', 'none')
    .style('opacity', 0.5)
}

function generateSeries(numPoints = 100) {
  const data = []
  for (let i = 0; i < numPoints; i++) {
    data.push(Math.sin(i * 0.2 + Math.random() * 0.5) + Math.random() * 0.4)
  }
  return data
}

function drawSeries() {
  // Ensure SVG exists (could be cleaned up)
  if (!svg) return;

  const x = d3.scaleLinear().domain([0, 80]).range([0, 300])
  const y = d3.scaleLinear().domain([-2, 2]).range([60, 0])
  const line = d3.line().x((d, i) => x(i)).y(d => y(d)).curve(d3.curveMonotoneX)

  const data = generateSeries()

  // Ensure SVG still exists before appending
  if (svg.node() === null) return; // Check if SVG was removed elsewhere

  const group = svg.append('g')
    .attr('transform', `translate(${Math.random() * window.innerWidth}, ${Math.random() * window.innerHeight}) rotate(${Math.random() * 360})`)

  const path = group.append('path')
    .datum(data)
    .attr('d', line)
    .attr('fill', 'none')
    .attr('stroke', 'white')
    .attr('stroke-width', 1.2)
    .attr('stroke-dasharray', function () {
      const length = this.getTotalLength()
      return `${length} ${length}`
    })
    .attr('stroke-dashoffset', function () {
      return this.getTotalLength()
    })

  path.transition()
    .duration(3000)
    .ease(d3.easeLinear)
    .attr('stroke-dashoffset', 0)
    .on('end', () => {
      // Check if group still exists before transitioning opacity
      if (group.node() !== null) {
          group.transition()
            .duration(2000)
            .style('opacity', 0)
            .remove() // Remove the group once faded out
      }
    })
}

function startAnimation() {
  // Check if already running (e.g., rapid slide changes)
  if (intervalId) return;
  createSvg()
  // Only start interval if SVG was successfully created
  if (svg) {
      intervalId = setInterval(drawSeries, 200)
  }
}

function stopAnimation() {
  if (intervalId) {
    clearInterval(intervalId)
    intervalId = null
  }
  if (svg) {
    // Stop any ongoing transitions on the SVG container itself
    // and its children before removing it
    svg.selectAll('*').interrupt(); // Stop transitions on children
    svg.interrupt().remove() // Stop transitions on SVG and remove
    svg = null
  }
}

// Hook into Slidev slide lifecycle
onSlideEnter(() => {
  requestAnimationFrame(() => {
      startAnimation()
  });
})

onSlideLeave(() => {
  stopAnimation()
})

// Clean up on component unmount (essential for SPA behavior)
onBeforeUnmount(() => {
  stopAnimation()
})

</script>

<template>
  <div ref="animatedBgEl" class="absolute inset-0 z-[-1]"></div>
</template>