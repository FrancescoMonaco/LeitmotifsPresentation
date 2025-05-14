<script setup>
import { onMounted, watch, ref, getCurrentInstance } from 'vue'
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

// Generate a unique ID for the container for this component instance
const instance = getCurrentInstance()
// Provides a unique ID for each component instance.
// Added a fallback just in case uid is not available in some edge case or older Vue setup.
const uniqueComponentId = instance && instance.uid !== undefined
                          ? instance.uid
                          : `fallback-${Math.random().toString(36).substring(2, 9)}`;
const containerId = ref(`d3-motifplot-${uniqueComponentId}`) // Use this ref for the div's ID

// Re-render if data or dimensions change
// Added props.width and props.height to dependencies, as changes should trigger a redraw.
watch(() => [props.timeSeries, props.motifs, props.width, props.height], () => {
  draw()
}, { deep: true }) // Consider deep watch if motif objects might change internally

// D3 drawing logic
function draw() {
  const containerSelector = `#${containerId.value}` // Use the unique ID
  const containerElement = d3.select(containerSelector)

  if (containerElement.empty()) {
    console.warn(`D3 container ${containerSelector} not found. Plotting will be skipped. This can happen if the component is rendered before its template div is in the DOM.`);
    return;
  }

  // Clear previous svg if exists
  containerElement.selectAll('*').remove()

  const margin = { top: 10, right: 10, bottom: 20, left: 30 } // Standard margins
  // Add axes (optional, but good practice)
  // const margin = { top: 20, right: 20, bottom: 30, left: 40 }; // Margins for axes


  const svg = containerElement
    .append('svg')
    .attr('width', props.width)
    .attr('height', props.height)

  if (!props.timeSeries || props.timeSeries.length === 0 || props.timeSeries[0].length === 0) {
    svg.append("text")
       .attr("x", props.width / 2)
       .attr("y", props.height / 2)
       .attr("text-anchor", "middle")
       .style("font-size", "16px")
       .text("No data to display.");
    return;
  }

  // Determine the number of time steps.
  // IMPORTANT: This assumes all series in props.timeSeries should have the same length,
  // or be plotted against the scale of the first series.
  // If series can have different lengths, you might want to use the max length:
  // const timeSteps = Math.max(...props.timeSeries.map(s => s.length));
  const timeSteps = props.timeSeries[0].length
  if (timeSteps === 0) {
      svg.append("text").attr("x", props.width / 2).attr("y", props.height / 2).attr("text-anchor", "middle").text("Empty series data.");
      return;
  }


  const xScale = d3.scaleLinear()
    .domain([0, timeSteps - 1])
    .range([margin.left, props.width - margin.right])

  const flatData = props.timeSeries.flat().filter(v => typeof v === 'number'); // Ensure only numbers for min/max
  let yDomain = d3.extent(flatData);

  if (yDomain[0] === undefined || yDomain[0] === yDomain[1]) { // Handle empty or single-point data
    const val = yDomain[0] === undefined ? 0 : yDomain[0];
    yDomain = [val - 1, val + 1]; // Default spread
     if (yDomain[0] === yDomain[1]) yDomain = [yDomain[0] - 0.5, yDomain[1] + 0.5]; // Ensure non-zero span
  }

  const yScale = d3.scaleLinear()
    .domain(yDomain)
    .range([props.height - margin.bottom, margin.top])


  const line = d3.line()
    .x((d, i) => xScale(i))
    .y(d => yScale(d))
    .curve(d3.curveMonotoneX)

  // Draw base series for each dimension
  props.timeSeries.forEach((series, dimIdx) => {
    // Ensure the series has the expected number of points for the current xScale, or handle appropriately.
    // For simplicity, this example assumes series data aligns with `timeSteps`.
    if (series && series.length > 0) {
        svg.append('path')
        .datum(series)
        .attr('fill', 'none')
        .attr('stroke', d3.schemeCategory10[dimIdx % 10])
        .attr('stroke-width', 1.5)
        .attr('d', line)
    }
  })

  // Draw motifs
  props.motifs.forEach((motif) => { // motifIdx is not used
    props.timeSeries.forEach((series, dimIdx) => {
      if (series && series.length > 0) {
          const highlightedData = series.map((val, i) =>
            (i >= motif.start && i <= motif.end && dimIdx !== 1) ? val : null // dimIdx != 1 is your original condition
          );

          // Only add path if there's something to highlight
          if (highlightedData.some(d => d !== null)) {
            svg.append('path')
            .datum(highlightedData)
            .attr('fill', 'none')
            .attr('stroke', motif.color || 'red')
            .attr('stroke-width', 3) // Make motifs more prominent
            .attr('d', d3.line()
                .defined(d => d !== null) // Important for drawing disconnected segments
                .x((d, i) => xScale(i))   // 'i' is the index in highlightedData, maps to time step
                .y(d => yScale(d))
                .curve(d3.curveMonotoneX)
            )
          }
      }
    })
  })
}

onMounted(() => {
  draw()
})
</script>

<template>
  <div :id="containerId" class="flex justify-center mt-4"></div> </template>