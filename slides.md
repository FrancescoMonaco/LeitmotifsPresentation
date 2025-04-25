---
# You can also start simply with 'default'
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: LEIT-motifs
info: |
  Scalable Motif Mining 
  in Multidimensional Time Series
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# open graph
# seoMeta:
#  ogImage: https://cover.sli.dev

---
<style>
@font-face {
  font-family: 'BPdots';
  src: url('/fonts/BPdotsUnicasePlus.otf') format('opentype');
  font-display: swap;
}

.leit-title {
  font-family: 'BPdots', sans-serif;
  font-size: 3rem;
  letter-spacing: 0.05em;
}
</style>
# <span class="leit-title">LEIT-motifs</span>

Scalable Motif Mining
in Multidimensional Time Series

---
transition: fade-out
---

# Basics
<div v-click>
We will define the following terms:

- Time Series
- Subsequence
- Motif

</div>

<v-click>

And look at the following concepts:


- How do we find motifs exactly?
- How do we find motifs approximately?

</v-click>
---
transition: fade-out
---

# What is a Time Series?


A **time series** is a sequence of numbers of length $n$, each representing a value at a specific time point.  
For example, daily temperatures recorded over a month form a time series.

<TimeSeriesAnimation />

---
transition: fade-out
---

# What is a Subsequence?
A **subsequence** is a portion of a time series, commonly referred to as a **window** of length $w$.
For example, a window of a day of temperatures is a subsequence of the time series of the month.

<Subsequence />

---
transition: fade-out
---

# What is a Motif?
A **motif** is a pair of subsequences that are similar to each other.

---
transition: fade-out
---

# The Multidimensional Case
All of the above definitions can be extended to the multidimensional case.

---
transition: fade-out
---

# How do we find motifs exactly?
The Matrix Profile Approach

<v-switch>
 <template #1> - The Matrix Profile is a data structure that allows us to find motifs in time series data efficiently. </template>
 <template #2> - It employs the **Cyclic Convolution Theorem** to compute in $O(n\log n)$ time the distances </template>
 <template #3> - It stores in $O(n)$ space the distance of each subsequence with its nearest neighbor </template>
</v-switch>

<script setup>
import MotifProfileAnimation from './components/MotifProfileAnimation.vue'

// Create random noise with two sinusoidal patterns
const data1D = Array.from({ length: 200 }, (_, i) => Math.random() * 2.3)
  .map((_, i) => {
    if (i % 20 < 10) return Math.sin(i * 0.2) + Math.random() * 0.3
    else return Math.cos(i * 0.2) + Math.random() * 0.3
  })
  .map((_, i) => {
    if (i % 20 < 10) return Math.sin(i * 0.2) + Math.random() * 0.3
    else return Math.cos(i * 0.2) + Math.random() * 0.3
  })


const dataMultiD = Array.from({ length: 100 }, (_, i) => [
  Math.sin(i * 0.2) + Math.random() * 0.3,
  Math.cos(i * 0.2) + Math.random() * 0.3,
])
</script>

<MotifProfileAnimation :data="data1D" :dims="1" :subseqLength="20" :speed="20" />


---
transition: fade-out
---

# How do we find motifs approximately?
The EMD Approach

---
transition: slide-left
---

# How do we find motifs approximately?
The Random Projection Approach

---
transition: fade-out
---

# A Hash Approach

---
transition: fade-out
---
 
# Details
<div v-click>
We will look at the different steps of the algorithm:

- Creating an hash index
- Finding the closest pairs
- Stopping once we are sure

</div>

---
transition: fade-out
---

# Hash Index
The Random Projection approach

We partition the space using random vectors and hash the data points into bins.
The hash function is defined as:
$$
    h(x) = \left\lfloor \frac{a\cdot x+b}{r} \right\rfloor
$$
where $a$ is a random vector, $b$ is a random number, and $r$ is the bin size.
---
transition: fade-out
---

# Closest Pairs

---
transition: fade-out
---

# A Stopping Criterion

$$
\text{Let } p(d) \text{ be the probability of sharing the same hash value at distance } d.
$$

$$
\text{Now, let } i \text{ be the number of concatenations and } j \text{ be the number of hash repetitions.} \\
\text{Then, the probability of sharing the same hash value is:} \\
  \left(1-p^i\right)^j 
$$


$$
      \left\{
        \begin{aligned}
        &\left(1-p^i\right)^j \quad &\textrm{ if }~~ i=K \\
        &\left(1-p^i\right)^j \cdot \left(1-p^{i+1}\right)^{L-j} \quad&\textrm{ otherwise}
        \end{aligned}
      \right.
$$

---
transition: fade-out
---

# A Running Example


---
layout: center
---