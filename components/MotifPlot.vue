<script setup>
import { onMounted, watch, ref } from 'vue' // Import ref
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

// Create a template ref for the container div
const plotContainer = ref(null)

// Re-render if data changes
watch(() => [props.timeSeries, props.motifs, props.width, props.height], () => {
  // Ensure the element ref is available before drawing
  if (plotContainer.value) {
    draw()
  }
}, { deep: true }) // Use deep watch if motifs array content might change internally without the array object itself changing


// D3 drawing logic
function draw() {
  // Ensure the container element exists
  if (!plotContainer.value) {
    console.error("MotifPlot container element not found.");
    return;
  }

  // Select the container using the template ref
  const container = d3.select(plotContainer.value);

  // Clear previous svg *within this specific container*
  container.selectAll('svg').remove() // More specific removal

  const margin = { top: 10, right: 10, bottom: 20, left: 30 }
  const svg = container // Append SVG to the ref'd container
    .append('svg')
    .attr('width', props.width)
    .attr('height', props.height)

  const timeSteps = props.timeSeries[0]?.length || 0; // Handle empty/invalid timeseries
  if (timeSteps === 0) return; // Don't draw if no data points

  const xScale = d3.scaleLinear()
    .domain([0, timeSteps - 1])
    .range([margin.left, props.width - margin.right])

  // Calculate global min/max across all dimensions
  const flatData = props.timeSeries.flat()
  if (flatData.length === 0) return; // Don't draw if no data

  const yDomain = d3.extent(flatData); // Use d3.extent for [min, max]
  // Provide fallback if min/max are undefined (e.g., all NaNs) or equal
  if (yDomain[0] === undefined || yDomain[1] === undefined) return;
  if (yDomain[0] === yDomain[1]) {
      yDomain[0] -= 1; // Add some padding if all values are the same
      yDomain[1] += 1;
  }

  const yScale = d3.scaleLinear()
    .domain(yDomain)
    .range([props.height - margin.bottom, margin.top])
    .nice() // Make the axis end on nice round values

  const line = d3.line()
    .x((d, i) => xScale(i))
    .y(d => yScale(d))
    .curve(d3.curveMonotoneX)

  // Draw base series for each dimension
  props.timeSeries.forEach((series, dimIdx) => {
    if (!series || series.length !== timeSteps) return; // Skip invalid series
    svg.append('path')
      .datum(series)
      .attr('fill', 'none')
      .attr('stroke', d3.schemeCategory10[dimIdx % 10])
      .attr('stroke-width', 1.5)
      .attr('d', line)
  })

  // Draw motifs
  // Filter motifs to be within data bounds
  const validMotifs = props.motifs.filter(motif =>
      motif.start >= 0 && motif.end < timeSteps && motif.start <= motif.end
  );

  const motifLine = d3.line() // Define motif line generator once
      .defined(d => d !== null) // Handle gaps created by filtering
      .x((d, i) => xScale(motif.start + i)) // Adjust index for sliced data
      .y(d => yScale(d))
      .curve(d3.curveMonotoneX)

  validMotifs.forEach((motif) => {
      props.timeSeries.forEach((series, dimIdx) => {
          if (!series || series.length !== timeSteps) return; // Skip invalid series

          // Extract only the relevant segment for the motif
          const highlightedSegment = series.slice(motif.start, motif.end + 1);

          // Skip drawing if the segment is empty
          if (highlightedSegment.length === 0) return;

          // Check if dimIdx != 1 condition is intended - it seems specific
          // If you want to highlight all dimensions for a motif, remove this check
          // const shouldHighlight = dimIdx !== 1; // Example: Original logic
          const shouldHighlight = true; // Example: Highlight all dimensions

          if (shouldHighlight) {
              svg.append('path')
                  .datum(highlightedSegment) // Use the sliced data
                  .attr('fill', 'none')
                  .attr('stroke', motif.color || 'red')
                  .attr('stroke-width', 3)
                  .attr('d', motifLine); // Apply the motif line generator
          }
      });
  });

  // Add Axes (Optional but good practice)
  const xAxis = d3.axisBottom(xScale);
  svg.append("g")
     .attr("transform", `translate(0,${props.height - margin.bottom})`)
     .call(xAxis);

  const yAxis = d3.axisLeft(yScale);
  svg.append("g")
     .attr("transform", `translate(${margin.left},0)`)
     .call(yAxis);
}

onMounted(() => {
  draw()
})

</script>

<template>
  <div ref="plotContainer" class="flex justify-center mt-4"></div>
</template>