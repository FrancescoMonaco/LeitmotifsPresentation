<template>
  <div class="animated-ts-container">
    <div class="animation-area">
      <svg :viewBox="`0 0 ${svgWidth} ${svgHeight}`" class="time-series-svg">
        <g :style="{ transform: `translateX(-${scrollOffset}px)` }">
          <path
            v-for="(wave, index) in waveforms"
            :key="`wave-${index}`"
            :d="wave.path"
            :stroke="wave.color"
            stroke-width="2"
            fill="none"
            class="waveform-path"
          />
        </g>
        <rect
          :x="windowPositionPx + scrollOffset"
          y="5"
          :width="windowWidthPx"
          :height="svgHeight - 10"
          class="sliding-window"
          :style="{ transform: `translateX(-${scrollOffset}px)` }"
        />
      </svg>

      <transition-group name="subsequence-anim" tag="div" class="subsequences-display"> 
      </transition-group>

      <transition name="codebook-anim">
        <div v-if="currentCodebook" class="codebook-display" :key="currentCodebook.id">
          <div class="codebook-label">Codebook:</div>
          <div class="hashes-container">
            <span
              v-for="(hash, hIndex) in currentCodebook.hashes"
              :key="hIndex"
              class="hash-block"
              :style="{ backgroundColor: hash.color }"
            >
              {{ hash.value }}
            </span>
          </div>
        </div>
      </transition>
    </div>

    <div class="controls-and-pq-area">
      <div class="priority-queue-display">
        <h4>Priority Queue (Top {{ pqMaxSize }})</h4>
        <transition-group name="pq-item" tag="ul" class="pq-list">
          <li v-for="item in priorityQueue" :key="item.id" class="pq-item">
            Pair ({{ item.cbIds[0].slice(-4) }}, {{ item.cbIds[1].slice(-4) }}) - W: {{ item.weight.toFixed(0) }}
          </li>
        </transition-group>
        <p v-if="!priorityQueue.length" class="pq-empty">Queue is empty.</p>
      </div>

      <div class="animation-controls">
        <button @click="toggleAnimation" :class="{ playing: isAnimating }">
          {{ isAnimating ? 'Pause' : 'Play' }}
        </button>
        <button @click="resetAnimation">Reset</button>
        <div class="status-log">
            <p>{{ currentStatus }}</p>
            <p v-if="lastWeight !== null">Last Weight: {{ lastWeight.toFixed(0) }} {{ lastWeightAction }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed, watch } from 'vue';

const svgWidth = 600;
const svgHeight = 130;
const numWaveforms = 3;
const waveformLength = 120; // Number of data points
const pointSpacing = 5; // Pixels between points
const totalWaveformWidth = waveformLength * pointSpacing;
const windowSizePoints = 10; // Window size in data points
const windowWidthPx = windowSizePoints * pointSpacing;
const scrollSpeed = 1; // pixels per frame for scrolling
const pqMaxSize = 3;
const D_THRESHOLD = 2; // Weight threshold for PQ

// --- Reactive State ---
const waveforms = ref([]);
const scrollOffset = ref(0);
const slidingWindowDataPosition = ref(0); // Position of window in terms of data points index
const isAnimating = ref(false);
const animationFrameId = ref(null);
const currentStatus = ref("Initialized. Press Play.");

const displayedSubsequences = ref([]); // { id, data, dimension }
const allCodebooks = ref([]); // { id, hashes: [{value, color}], originalSubsequenceIds }
const currentCodebook = ref(null);
const priorityQueue = ref([]); // { id, cbIds: [id1, id2], weight }
const lastWeight = ref(null);
const lastWeightAction = ref("");

let nextSubsequenceId = 0;
let nextCodebookId = 0;
let nextPqItemId = 0;

// --- Computed Properties ---
const windowPositionPx = computed(() => slidingWindowDataPosition.value * pointSpacing);

// --- Waveform Generation ---
function generateWaveformPath(data, index) {
  const waveHeight = svgHeight / (numWaveforms + 1);
  const yOffset = waveHeight * (index + 1);
  let path = `M 0 ${yOffset}`;
  data.forEach((point, i) => {
    const x = i * pointSpacing;
    const y = yOffset - point * (waveHeight * (0.1*(index+1))); // Scale point to fit wave area
    path += ` L ${x} ${y}`;
  });
  return path;
}

function generateInitialWaveforms() {
  const colors = ['#FF7F50', '#00FF7F', '#afaaff'];
  waveforms.value = Array.from({ length: numWaveforms }, (_, i) => {
    const data = Array.from({ length: waveformLength }, () => (Math.random() - 0.5) * 2 + (-0.5*i)); // Random data
    return {
      id: `w${i}`,
      color: colors[i % colors.length],
      data: data,
      path: generateWaveformPath(data, i),
    };
  });
}

// --- Animation Loop ---
let lastExtractionTime = 0;
const extractionInterval = 1000; // ms, roughly how often to extract

function animate(timestamp) {
  if (!isAnimating.value) return;

  scrollOffset.value += scrollSpeed;
  if (scrollOffset.value + svgWidth > totalWaveformWidth) {
    scrollOffset.value = 0; // Loop scrolling
    // Could also move window back to start or implement other logic
  }

  // Move sliding window based on scroll (or independently if desired)
  // For this demo, let's tie it to the scroll for simplicity, effectively making it "scan"
  const effectiveWindowStartPx = windowPositionPx.value - scrollOffset.value;
  if (effectiveWindowStartPx < -windowWidthPx * 0.8 ) { // If window moved mostly off-screen left due to scroll
     slidingWindowDataPosition.value += (waveformLength * 0.2); // Jump it forward
     if (slidingWindowDataPosition.value >= waveformLength - windowSizePoints) {
        slidingWindowDataPosition.value = 0;
     }
  } else if (effectiveWindowStartPx > svgWidth * 0.7) { // if window moves too far right on screen
    // For this demo, we let scrollOffset reset handle it.
    // Or, reset slidingWindowDataPosition.value = scrollOffset.value / pointSpacing;
  }


  // --- Subsequence Extraction, Hashing, Weight, PQ ---
  if (timestamp - lastExtractionTime > extractionInterval) {
    lastExtractionTime = timestamp;
    currentStatus.value = "Extracting subsequences...";

    // 1. Subsequence Extraction
    const extracted = [];
    waveforms.value.forEach((wave, dimIndex) => {
      const start = Math.floor((scrollOffset.value + windowWidthPx * 0.1) / pointSpacing) + slidingWindowDataPosition.value; // Approx start point in data
      const end = start + windowSizePoints;
      if (end <= wave.data.length) {
        extracted.push({
          id: `sub-${nextSubsequenceId++}`,
          data: wave.data.slice(start, end),
          dimension: dimIndex,
        });
      }
    });

    if (extracted.length === numWaveforms) {
      displayedSubsequences.value = extracted; // Trigger visual update

      setTimeout(() => {
        currentStatus.value = "Hashing subsequences...";
        // 2. Hashing Phase (Simplified)
        const hashes = displayedSubsequences.value.map((sub, i) => ({
            value: `${Math.random().toString(10).slice(2, 6)}`,
            color: waveforms.value[i % waveforms.value.length].color + 'AA' // Semi-transparent waveform color
        }));
        const newCb = {
          id: `${nextCodebookId++}`,
          hashes: hashes,
          originalSubsequenceIds: displayedSubsequences.value.map(s => s.id)
        };
        currentCodebook.value = newCb; // Display current codebook
        allCodebooks.value.push(newCb);

        displayedSubsequences.value = []; // Clear for next extraction

        // 3. Weight Computation (Simplified)
        if (allCodebooks.value.length >= 2) {
          currentStatus.value = "Computing weight...";
          const cb1 = allCodebooks.value[allCodebooks.value.length - 2];
          const cb2 = allCodebooks.value[allCodebooks.value.length - 1];
          const weight = Math.trunc(Math.random()*4); // Abstract w()
          lastWeight.value = weight;

          // 4. Priority Queue Insertion
          if (weight >= D_THRESHOLD) {
            lastWeightAction.value = `> ${D_THRESHOLD.toFixed(0)} -> Added to PQ.`;
            currentStatus.value = "Adding to Priority Queue...";
            priorityQueue.value.unshift({ id: `pq-${nextPqItemId++}`, cbIds: [cb1.id, cb2.id], weight });
            priorityQueue.value.sort((a, b) => b.weight - a.weight); // Keep sorted
            if (priorityQueue.value.length > pqMaxSize) {
              priorityQueue.value.pop(); // Remove lowest if full
            }
          } else {
            lastWeightAction.value = `<= ${D_THRESHOLD.toFixed(0)} -> Discarded.`;
            currentStatus.value = "Weight below threshold.";
          }
        }
         // Shift window slightly after an extraction for visual feedback
        slidingWindowDataPosition.value = (slidingWindowDataPosition.value + windowSizePoints / 2) % (waveformLength - windowSizePoints);

      }, 700); // Delay for hashing viz
    }
  }

  animationFrameId.value = requestAnimationFrame(animate);
}

// --- Controls ---
function toggleAnimation() {
  isAnimating.value = !isAnimating.value;
  if (isAnimating.value) {
    currentStatus.value = "Animation Playing...";
    lastExtractionTime = performance.now(); // Reset timer on play
    animationFrameId.value = requestAnimationFrame(animate);
  } else {
    currentStatus.value = "Animation Paused.";
    if (animationFrameId.value) {
      cancelAnimationFrame(animationFrameId.value);
    }
  }
}

function resetAnimation() {
  isAnimating.value = false;
  if (animationFrameId.value) {
    cancelAnimationFrame(animationFrameId.value);
  }
  scrollOffset.value = 0;
  slidingWindowDataPosition.value = 0; // Initial position
  generateInitialWaveforms();
  displayedSubsequences.value = [];
  allCodebooks.value = [];
  currentCodebook.value = null;
  priorityQueue.value = [];
  lastWeight.value = null;
  lastWeightAction.value = "";
  currentStatus.value = "Animation Reset. Press Play.";
  nextSubsequenceId = 0;
  nextCodebookId = 0;
  nextPqItemId = 0;
}

onMounted(() => {
  generateInitialWaveforms();
  slidingWindowDataPosition.value = 0; // Initial position
});

onUnmounted(() => {
  isAnimating.value = false;
  if (animationFrameId.value) {
    cancelAnimationFrame(animationFrameId.value);
  }
});

</script>

<style scoped>
.animated-ts-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px;
  font-family: sans-serif;
  background-color: #282c34; /* Darker background */
  color: #f0f0f0;
  padding: 15px;
  border-radius: 8px;
  width: 100%;
  max-width: 700px; /* Control max width */
  margin: auto;
}

.animation-area {
  width: 100%;
  position: relative; /* For absolute positioning of overlays */
  background-color: #333a4500;
  padding: 10px;
  border-radius: 6px;
  min-height: 200px; /* To accommodate visuals */
}

.time-series-svg {
  width: 100%;
  height: auto; /* Maintain aspect ratio */
  border: 1px solid #444c56;
  border-radius: 4px;
  overflow: hidden; /* Crucial for scrolling effect */
}

.waveform-path {
  transition: d 0.1s linear; /* If paths were to change dramatically */
}

.sliding-window {
  fill: rgba(0, 123, 255, 0.25);
  stroke: rgba(0, 123, 255, 0.8);
  stroke-width: 1.5;
  /* No transition here as x is frequently updated */
}

.subsequences-display {
  position: absolute;
  top: 10px;
  right: 10px;
  display: flex;
  flex-direction: column;
  gap: 5px;
  z-index: 10;
}

.subsequence-box {
  background-color: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(3px);
  color: #fff;
  padding: 4px 8px;
  border-radius: 3px;
  font-size: 0.75em;
  border-left-width: 4px;
  border-left-style: solid;
  box-shadow: 0 1px 3px rgba(0,0,0,0.3);
}

.subsequence-anim-enter-active,
.subsequence-anim-leave-active {
  transition: all 0.5s ease;
}
.subsequence-anim-enter-from,
.subsequence-anim-leave-to {
  opacity: 0;
  transform: translateX(20px);
}

.codebook-display {
  position: absolute;
  bottom: -15px;
  left: 30px;
  background-color: rgba(0, 0, 0, 0.4);
  backdrop-filter: blur(3px);
  padding: 8px;
  border-radius: 4px;
  z-index: 10;
  display: flex;
  flex-direction: column;
  gap: 4px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.4);
}
.codebook-label {
    font-size: 0.8em;
    color: #aaa;
    margin-bottom: 2px;
}
.hashes-container {
  display: flex;
  gap: 4px;
}
.hash-block {
  padding: 3px 5px;
  border-radius: 2px;
  font-size: 0.7em;
  color: #111; /* Darker text on colored background */
  font-family: monospace;
  min-width: 30px;
  text-align: center;
}

.codebook-anim-enter-active {
  transition: all 0.5s ease-out;
}
.codebook-anim-leave-active {
  transition: all 0.3s ease-in;
}
.codebook-anim-enter-from {
  opacity: 0;
  transform: translateY(20px) scale(0.8);
}
.codebook-anim-leave-to {
  opacity: 0;
  transform: scale(0.7);
}


.controls-and-pq-area {
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: flex-start; /* Align top */
  gap: 15px;
  margin-top:10px;
}

.animation-controls {
  display: flex;
  flex-direction: column; /* Stack buttons and status */
  gap: 8px;
  align-items: flex-start;
  background-color: #333a4500;
  padding: 10px;
  border-radius: 6px;     
  flex-grow:1; /* Take remaining space */
}
.animation-controls button {
  padding: 8px 12px;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s ease;
  min-width: 80px;
}
.animation-controls button.playing {
  background-color: #ffc107;
  color: #333;
}
.animation-controls button:hover {
  opacity: 0.8;
}
.status-log {
    font-size: 0.8em;
    color: #ccc;
    min-height: 30px;
}
.status-log p {
    margin: 2px 0;
}


.priority-queue-display {
  width: 45%; /* Fixed width for PQ */
  min-width: 220px;
  background-color: #333a4500;
  padding: 10px;
  border-radius: 6px;
}
.priority-queue-display h4 {
  margin-top: 0;
  margin-bottom: 8px;
  font-size: 0.9em;
  color: #00acc1;
  text-align: center;
}
.pq-list {
  list-style: none;
  padding: 0;
  margin: 0;
  max-height: 120px; /* Control height */
  overflow-y: auto;
}
.pq-item {
  background-color: #4f5b6a;
  color: #e0e0e0;
  padding: 5px 8px;
  margin-bottom: 4px;
  border-radius: 3px;
  font-size: 0.75em;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  box-shadow: inset 0 0 3px rgba(0,0,0,0.2);
}
.pq-empty {
    font-size: 0.8em;
    text-align: center;
    color: #777;
}

.pq-item-enter-active,
.pq-item-leave-active {
  transition: all 0.5s cubic-bezier(0.55, 0, 0.1, 1);
}
.pq-item-enter-from {
  opacity: 0;
  transform: scaleY(0) translateY(-15px);
}
.pq-item-leave-to {
  opacity: 0;
  transform: translateX(20px) scaleX(0.5);
}
.pq-item-move {
  transition: transform 0.5s ease;
}

/* Scrollbar styling for PQ */
.pq-list::-webkit-scrollbar {
  width: 6px;
}
.pq-list::-webkit-scrollbar-track {
  background: #333a45;
  border-radius: 3px;
}
.pq-list::-webkit-scrollbar-thumb {
  background: #5f6b7c;
  border-radius: 3px;
}
.pq-list::-webkit-scrollbar-thumb:hover {
  background: #7f8ba0;
}
</style>