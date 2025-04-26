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
A **subsequence** $T_a,w$ is a portion of a time series starting at point $a$ and of length $w$, $w$ isncommonly referred to as the <span v-mark.red="1">**window** </span> size.
For example, a window of a day of temperatures is a subsequence of the time series of the month.

<Subsequence />

---
transition: fade-out
---

# What is a Motif?
A **motif** is a pair of subsequences $T_a$, $T_b$ that are <span v-mark.red="1">the most similar </span> to each other.

<MotifPlot
  :time-series="[
    [0,1,2,3,2,1,0,4,5,4,3,2,8,4,5,6,7,4,3,3,4,5,4,3]
  ]"
  :dimension="1"
  :motifs="[
    { start: 1, end: 5, color: 'red' },
    { start: 19, end: 25, color: 'red' }]"
  :width="700"
  :height="250"
/>

---
transition: fade-out
---

# What do we mean by similar?
The z-normalized Euclidean distance

By similar we mean that the <span v-mark.circle.red="1"> shapes </span> of the subsequences are similar.

A common way to measure this is the **z-normalized Euclidean distance**:
$$
    d(T_a, T_b) = \sqrt{\sum_{i=0}^{w-1} \left(\frac{T_a[i] - \bar{T_a}}{\sigma_{T_a}} - \frac{T_b[i] - \bar{T_b}}{\sigma_{T_b}}\right)^2}
$$

where $\bar{T_a}$ and $\sigma_{T_a}$ are the mean and standard deviation of the subsequence $T_a$.


Other distance measures can be used, such as the **Dynamic Time Warping** (DTW) distance.

---
transition: fade-out
---

# The Multidimensional Case
All of the above definitions can be extended to the multidimensional case.

A **multidimensional time series** is a collection of $D$ vectors of length $n$ where each vector represents a different dimension of the time series.

For example, a time series of temperature and humidity recorded over a month is a multidimensional time series.

<MultiTimeSeries />

---
transition: fade-out
---

# A Multidimensional Motif

---
transition: fade-out
---

# Problem Definition
Top $k$ motif discovery

Given a multidimensional time series $T$ of length $n$ and dimensionality $D$, a window size $w$, a number of motifs $k$, and motif dimensionality $d$, the problem is to find the top $k$ motifs of dimensionality $d$ in the time series.


- We have to find the subset of dimensions that span the motif

---
transition: fade-out
---

# How do we find motifs exactly?
The **Matrix Profile** Approach

<v-switch>
 <template #1> - The Matrix Profile is a data structure that allows us to find motifs in time series data efficiently. </template>
 <template #2> - It employs the **Cyclic Convolution Theorem** to compute in $O(n\log n)$ time the distances </template>
 <template #3> - It stores in $O(n)$ space the distance of each subsequence with its nearest neighbor </template>
 <template #4> -  </template>
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
The **EMD** Approach

A meta time series is obtained by applying **Principal Component Analysis** to the multidimentional time series. 

Motifs are then discovered with the **Minimum Description Length** principle.

Pros:
- The dimensionality of the time series does not have an impact on the discovery

Cons:
-  Motifs may be masked by useless dimensions or noise
- Additional step to retrieve the dimensions that span the motif

---
transition: slide-left
---

# How do we find motifs approximately?
The **Random Selection** Approach

We randomly pick **combinations** of the dimensions of the time series

Pros:
- Noise or irrelevant dimensions do not impact the discovery phase
- We know the dimensions that span the motif

Cons:
- Very complex way to find motifs
- In practice the quality of the results is bad

---
transition: fade-out
---

# A Hash Approach
The general framework

Ideally we want to use an hash function that can 
tell us what are the colliding subsequences 
and what are the dimensions where they collide.

Then we want to compute the distances of these collisions and we want to stop when the result is good for our user.



---
transition: fade-out
---
 
# Details
<div v-click>
We will look at the different steps of the algorithm:

- Creating an hash index
- Finding the closest pairs
- Stopping once we are sure the result is _good enough_

</div>

---
transition: fade-out
---

# Hash Index
Random Projections

We partition the space using random vectors and hash the data points into bins.
The hash function is defined as:
$$
    h(x) = \left\lfloor \frac{a\cdot x+b}{r} \right\rfloor
$$
where $a$ is a random vector, $b$ is a random number, and $r$ is the bin size.

<RandomProjection />
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
given time series $T$, window $w$, number of motifs to find $k$ and motif dimensionality $d$

---
transition: fade-out
---

# Optimizations
Some technical details

---
transition: fade-out
---

# Finding motifs of different dimensionalities
Overcoming the need of $d$ in input