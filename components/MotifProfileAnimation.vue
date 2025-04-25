<script setup>
import { onMounted, ref } from 'vue'
import * as d3 from 'd3'
import { Play, Pause, RotateCcw } from 'lucide-vue-next'

const props = defineProps({
  data: Array,
  dims: Number,
  subseqLength: { type: Number, default: 10 },
  speed: { type: Number, default: 20 }
})

const timeSeries = props.data
const subseqLength = props.subseqLength
const dims = props.dims

const isPlaying = ref(false)
let probeIndex = 0
let animationTimeout

function reset() {
  probeIndex = 0;
  // Update this line:
  overallBestMatch = { probeIdx: -1, matchIdx: -1, dist: Infinity };
  profilePoints.length = 0;
  // Need to clear motifPath explicitly if reset is hit mid-animation or after completion
  if (motifPath) motifPath.selectAll('*').remove();
  // Clear other paths too
  if (probePath) probePath.datum([]).attr('d', null);
  if (matchPath) matchPath.datum([]).attr('d', null);
  if (profileLine) profileLine.datum([]).attr('d', null);
  if (distanceText) distanceText.style('display', 'none');
  // Re-setup if needed, or ensure existing elements are cleared/reset
  if (svg) {
      svg.selectAll('path.data-line').remove(); // Add a class to the base line if needed
      svg.selectAll('path.probe-path').remove(); // Add classes to make selection easier
      svg.selectAll('path.match-path').remove();
      svg.selectAll('path.motif-highlight').remove();
      svg.selectAll('path.profile-line').remove();
      svg.selectAll('text.distance-text').remove();
      svg.remove(); // Or just clear contents
  }
  setupBase(); // Recreate SVG and elements
}

function togglePlay() {
  if (isPlaying.value) {
    clearTimeout(animationTimeout)
    isPlaying.value = false
  } else {
    isPlaying.value = true
    animate()
  }
}

let svg, xScale, yScale, profileYScale, probePath, matchPath, motifPath, profileLine, distanceText
let overallBestMatch = { probeIdx: -1, matchIdx: -1, dist: Infinity };
const profilePoints = []
const isMultidim = dims > 1
const flattened = isMultidim ? timeSeries.map(row => row[0]) : timeSeries

function setupBase() {
  const width = 700
  const height = 230
  const profileHeight = 70
  const margin = { top: 20, right: 20, bottom: 30, left: 30 }

  svg = d3
    .select('#motif-vis')
    .html('')
    .append('svg')
    .attr('width', width)
    .attr('height', height + profileHeight + 50)

  xScale = d3.scaleLinear().domain([0, timeSeries.length - 1]).range([margin.left, width - margin.right])
  yScale = d3.scaleLinear().domain([-3, 3]).range([height - margin.bottom, margin.top])
  profileYScale = d3.scaleLinear().domain([0, 5]).range([height + profileHeight, height + 20])

  const baseLine = d3.line()
    .x((_, i) => xScale(i))
    .y(d => yScale(d))
    .curve(d3.curveMonotoneX)

  svg.append('path')
    .datum(flattened)
    .attr('fill', 'none')
    .attr('stroke', '#ccc')
    .attr('stroke-width', 2)
    .attr('d', baseLine)

  probePath = svg.append('path').attr('fill', 'none').attr('stroke', 'orange').attr('stroke-width', 3)
  matchPath = svg.append('path').attr('fill', 'none').attr('stroke', '#ff9800').attr('stroke-width', 2)
  motifPath = svg.append('g')
  profileLine = svg.append('path').attr('fill', 'none').attr('stroke', 'steelblue').attr('stroke-width', 2)

  distanceText = svg.append('text')
    .attr('fill', '#444')
    .attr('font-size', '14px')
    .attr('text-anchor', 'middle')
    .style('display', 'none')
}

function euclideanDistance(a, b) {
  let sum = 0
  for (let i = 0; i < a.length; i++) {
    if (dims === 1) {
      const diff = a[i] - b[i]
      sum += diff * diff
    } else {
      for (let d = 0; d < dims; d++) {
        const diff = a[i][d] - b[i][d]
        sum += diff * diff
      }
    }
  }
  return sum
}

function animate() {
  if (!isPlaying.value) return
  if (probeIndex >= timeSeries.length - subseqLength) { // Use >= for clarity
     // Clear temporary paths used during animation
     probePath.style('display', 'none'); // Hide instead of removing if needed later
     matchPath.style('display', 'none');
     distanceText.style('display', 'none');
     motifPath.selectAll('*').remove(); // Clear previous motif attempts if any

     // Check if a valid motif was found
     if (overallBestMatch.probeIdx !== -1 && overallBestMatch.matchIdx !== -1) {
         console.log(`Motif found: Index ${overallBestMatch.probeIdx} <-> Index ${overallBestMatch.matchIdx} (Dist: ${Math.sqrt(overallBestMatch.dist).toFixed(2)})`); // Log actual distance

         // Draw the two subsequences of the motif
         [overallBestMatch.probeIdx, overallBestMatch.matchIdx].forEach((start) => {
             const motifSubsequence = timeSeries.slice(start, start + subseqLength);
             const motifSubsequenceFlat = isMultidim ? motifSubsequence.map(v => v[0]) : motifSubsequence;

             motifPath.append('path')
                 .datum(motifSubsequenceFlat)
                 .attr('class', 'motif-highlight') // Add class for easier selection/styling
                 .attr('fill', 'none')
                 .attr('stroke', 'firebrick')
                 .attr('stroke-width', 3)
                 .attr('d', d3.line()
                     .x((_, i) => xScale(start + i))
                     .y(d => yScale(d))
                     .curve(d3.curveMonotoneX)
                 );
         });
     } else {
          console.log("No motif found (or animation didn't run long enough).");
     }

     isPlaying.value = false;
     clearTimeout(animationTimeout); // Ensure timeout is cleared
     return; // End animation
 }

  const current = timeSeries.slice(probeIndex, probeIndex + subseqLength)
  const currentFlat = isMultidim ? current.map(v => v[0]) : current

  probePath.datum(currentFlat)
    .attr('d', d3.line()
      .x((_, i) => xScale(probeIndex + i))
      .y(d => yScale(d))
      .curve(d3.curveMonotoneX))

  let minDist = Infinity
  let matchIdx = 0
  for (let i = 0; i < timeSeries.length - subseqLength; i++) {
    if (Math.abs(i - probeIndex) < subseqLength) continue
    const candidate = timeSeries.slice(i, i + subseqLength)
    const dist = euclideanDistance(current, candidate)
    if (dist < minDist) {
      minDist = dist
      matchIdx = i
    }
  }

  const match = timeSeries.slice(matchIdx, matchIdx + subseqLength)
  const matchFlat = isMultidim ? match.map(v => v[0]) : match

  matchPath.datum(matchFlat)
    .attr('d', d3.line()
      .x((_, i) => xScale(matchIdx + i))
      .y(d => yScale(d))
      .curve(d3.curveMonotoneX))

  const centerX = xScale(matchIdx + subseqLength / 2)
  const centerY = d3.mean(matchFlat.map(yScale))
  distanceText.style('display', null).attr('x', centerX).attr('y', centerY - 10).text(`dist = ${minDist.toFixed(2)}`)

  if (minDist < overallBestMatch.dist) {
    overallBestMatch = { probeIdx: probeIndex, matchIdx: matchIdx, dist: minDist };
}

  profilePoints.push(minDist)
  profileLine.datum(profilePoints)
    .attr('d', d3.line()
      .x((_, i) => xScale(i))
      .y(d => profileYScale(d))
      .curve(d3.curveMonotoneX))

  probeIndex++
  animationTimeout = setTimeout(animate, props.speed)
}

onMounted(() => {
  setupBase()
})
</script>

<template>
  <div class="flex flex-col items-center mt-4">
    <div class="mb-2 space-x-4">
      <button @click="togglePlay" class="bg-blue-500 text-white p-2 rounded-full">
        <component :is="isPlaying ? Pause : Play" class="w-5 h-5" />
      </button>
      <button @click="reset" class="bg-gray-500 text-white p-2 rounded-full">
        <RotateCcw class="w-5 h-5" />
      </button>
    </div>
    <div id="motif-vis"></div>
  </div>
</template>
