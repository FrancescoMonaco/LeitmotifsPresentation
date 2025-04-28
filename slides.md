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
transition: fade
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
transition: fade
---

# What is a Time Series?


A **time series** is a sequence of numbers of length $n$, each representing a value at a specific time point.  
For example, daily temperatures recorded over a month form a time series.

<TimeSeriesAnimation />

---
transition: fade
---

# What is a Subsequence?
A **subsequence** $T_a,w$ is a portion of a time series starting at point $a$ and of length $w$, $w$ isncommonly referred to as the <span v-mark.red="1">**window** </span> size.
For example, a window of a day of temperatures is a subsequence of the time series of the month.

<Subsequence />

---
transition: fade
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
transition: fade
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
transition: fade
---

# The Multidimensional Case
All of the above definitions can be extended to the multidimensional case.

A **multidimensional time series** is a collection of $D$ vectors of length $n$ where each vector represents a different dimension of the time series.

For example, a time series of temperature and humidity recorded over a month is a multidimensional time series.

<MultiTimeSeries />

---
transition: fade
---

# A Multidimensional Motif

---
transition: fade
---

# Problem Definition
Top $k$ motif discovery

Given a multidimensional time series $T$ of length $n$ and dimensionality $D$, a window size $w$, a number of motifs $k$, and motif dimensionality $d$, the problem is to find the top $k$ motifs of dimensionality $d$ in the time series.


- We have to find the subset of dimensions that span the motif

---
transition: fade
---

# How do we find motifs exactly?
The **Matrix Profile** Approach

 - The Matrix Profile is a data structure that allows us to find motifs in time series data efficiently.
---
transition: fade
---

# How do we find motifs exactly?
The **Matrix Profile** Approach

 - The Matrix Profile is a data structure that allows us to find motifs in time series data efficiently.
 - It employs the **Cyclic Convolution Theorem** to compute in $O(n\log n)$ time the distances

---
transition: fade
---

# How do we find motifs exactly?
The **Matrix Profile** Approach

 - The Matrix Profile is a data structure that allows us to find motifs in time series data efficiently.
 - It employs the **Cyclic Convolution Theorem** to compute in $O(n\log n)$ time the distances
 - It stores in $O(n)$ space the distance of each subsequence with its nearest neighbor

---
transition: fade
---

# How do we find motifs exactly?
The **Matrix Profile** Approach

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
transition: fade
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
transition: fade
---

# How do we find motifs approximately?
The **EMD** Approach

A meta time series is obtained by applying **Principal Component Analysis** to the multidimentional time series. 

Motifs are then discovered with the <span v-mark.circle.green="0"> Minimum Description Length </span> principle.

![](/images/mdl1.png)

---
transition: fade
---

# How do we find motifs approximately?
The **EMD** Approach

A meta time series is obtained by applying **Principal Component Analysis** to the multidimentional time series. 

Motifs are then discovered with the <span v-mark.circle.green="0"> Minimum Description Length </span> principle.

![](/images/mdl2.png)

---
transition: fade
---

# How do we find motifs approximately?
The **EMD** Approach

A meta time series is obtained by applying **Principal Component Analysis** to the multidimentional time series. 

Motifs are then discovered with the <span v-mark.circle.green="0"> Minimum Description Length </span> principle.

![](/images/mdl3.png)

---
transition: slide-left
---

# How do we find motifs approximately?
The **Random Selection** Approach

We randomly pick **combinations** of the dimensions of the time series, symbolize them and fill a collision matrix each time we find a collision.

Pros:
- Noise or irrelevant dimensions do not impact the discovery phase
- We know the dimensions that span the motif

Cons:
- Very complex way to find motifs
- In practice the quality of the results is bad

---
transition: fade
layout: two-cols
---

# A Hash Approach
The general framework

- Ideally we want to use an hash function that can 
tell us what are the colliding subsequences 
and what are the dimensions where they collide.

- Then we want to compute the distances of these collisions and we want to stop when the result is good for our user.


---
transition: fade
layout: center
---
 
# Details


---
transition: fade
---
 
# Details
We will look at the different steps of the algorithm:

- <span v-mark.green="1"> Creating an hash index </span> of the time series
- <span v-mark.green="2"> Finding the closest pairs </span> in the hash index
- <span v-mark.green="3"> Stopping once we are sure the result is _good enough_ </span>

---
transition: fade
layout: two-cols
---

# Hash Index
Random Projections

We partition the space using random vectors and hash the data points into bins.
The hash function is defined as:
$$
    h(x) = \left\lfloor \frac{a\cdot x+b}{r} \right\rfloor
$$
where $a$ is a random vector, $b$ is a random number, and $r$ is the bin size.

::right::

<RandomProjection />

---
transition: fade
layout: two-cols
---

# Hash Index
Random Projections

We partition the space using random vectors and hash the data points into bins.
The hash function is defined as:
$$
    h(x) = \left\lfloor \frac{a\cdot x+b}{r} \right\rfloor
$$
where $a$ is a random vector, $b$ is a random number, and $r$ is the bin size.

::right::  
<br>

<br>

<br>

 - In particular, we hash each dimension of the subsequences independently.

---
transition: fade
layout: two-cols
---

# Hash Index
Random Projections

We partition the space using random vectors and hash the data points into bins.
The hash function is defined as:
$$
    h(x) = \left\lfloor \frac{a\cdot x+b}{r} \right\rfloor
$$
where $a$ is a random vector, $b$ is a random number, and $r$ is the bin size.

::right::  
<br>

<br>

<br>

 - In particular, we hash each dimension of the subsequences independently.
 - We end up with a codebook of $d$ hash values for each subsequence.

---
transition: fade
---

# Closest Pairs
Weighting the collisions

Once we have built the hash index, we can scan the bins and look for colliding subsequences.

In particular we track the <span v-mark.green="1"> weight </span> of the collisions, which is defined as:
$$
    w(T_a, T_b) = \sum_{i=0}^{d-1} \left(h(T_a[i]) \wedge h(T_b[i])\right)
$$


where $h(T_a[i])$ is the hash value of the $i$-th dimension of the subsequence $T_a$ and $\wedge$ is the AND operator.
<br>



---
transition: fade
---

# Closest Pairs
Weighting the collisions

Once we have built the hash index, we can scan the bins and look for colliding subsequences.

In particular we track the <span v-mark.green="0"> weight </span> of the collisions, which is defined as:
$$
    w(T_a, T_b) = \sum_{i=0}^{d-1} \left(h(T_a[i]) \wedge h(T_b[i])\right)
$$


where $h(T_a[i])$ is the hash value of the $i$-th dimension of the subsequence $T_a$ and $\wedge$ is the AND operator.
<br>

We then compute the distance of the pairs of <span v-mark.green="0"> subsequences whose weight is $>d$ </span>, the requested motif dimensionality.
---
transition: fade
---

# A Stopping Criterion

$$
\text{Let } p(d) \text{ be the probability of sharing the same hash value at distance } d.
$$

---
transition: fade
---

# A Stopping Criterion

$$
\text{Let } p(d) \text{ be the probability of sharing the same hash value at distance } d.
$$
<br>
$$
\text{Now, let } i \text{ be the number of the current concatenation and } j \text{ be the number of the hash repetition.} \\
\text{Then, the probability of sharing the same hash value is:} \\
  \left(1-p^i\right)^j 
$$

---
transition: fade
---

# A Stopping Criterion

$$
\text{Let } p(d) \text{ be the probability of sharing the same hash value at distance } d.
$$
<br>
$$
\text{Now, let } i \text{ be the number of the current concatenation and } j \text{ be the number of the hash repetition.} \\
\text{Then, the probability of sharing the same hash value is:} \\
  \left(1-p^i\right)^j 
$$
<br>

$$
\text{Using the approach of PUFFINN*, we can define the probability of sharing the same hash value as:} \\
      \left\{
        \begin{aligned}
        &\left(1-p^i\right)^j \quad &\textrm{ if }~~ i=K \\
        &\left(1-p^i\right)^j \cdot \left(1-p^{i+1}\right)^{L-j} \quad&\textrm{ otherwise}
        \end{aligned}
      \right.
      \\
\text{with } K \text{ and } L \text{ being the maximum number of concatenations and repetitions, respectively.}
$$

###### *([Aumüller et al., 2020](https://arxiv.org/abs/1906.12211))
---
transition: fade
---

# A Running Example
given time series $T$, window $w$, number of motifs to find $k$ and motif dimensionality $d$

---
transition: fade
---

# Experimental Results
The datasets

---
transition: fade
---

# Experimental Results
Time to find the top motif

---
transition: fade
layout: center
---

# Optimizations
Some technical details

---
transition: fade
---

# Finding motifs of different dimensionalities
Overcoming the need of $d$ in input