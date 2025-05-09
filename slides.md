---
# You can also start simply with 'default'
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
#background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: LEIT-motifs
titleTemplate: '%s'
author: 'Francesco Pio Monaco'
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

<div class="absolute bottom-10">
  <span class="font-700">
    Francesco Pio Monaco, May 2025
  </span>
</div>

<TSBackground />

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
A **subsequence** $T_{a,w}$ is a portion of a time series starting at point $a$ and of length $w$, $w$ is commonly referred to as the <span v-mark.red="1">**window** </span> size.
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

By similar we mean that the <span v-mark.circle.red="1"> shapes </span> of the subsequences resemble each other.

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
Notice how one of the dimensions is just noise

<MotifPlot
  :time-series="[
    [0,1,2,3,2,1,0,4,5,4,3,2,6,4,5,6,7,4,3,3,4,5,4,3],
    [7.1,7.5,7.5,7.5,7.2,7.3, 7.1, 7.5, 7.3, 7.2, 7.13, 7.45, 7.23, 7.11, 7.12, 7.43, 7.34, 7.1,7.04, 7.23, 7.12, 7.32, 7.5,7.5,7.5,7.2],
    [10, 23, 12, 13, 12, 11, 10, 14, 15, 14, 13, 12, 18, 14, 15, 16, 17, 14, 13,  15, 10, 11, 12, 11]
  ]"
  :dimension="3"
  :motifs="[
    { start: 1, end: 5, color: 'red' },
    { start: 19, end: 25, color: 'red' }]"
  :width="700"
  :height="400"
/>

---
transition: fade
---

# Problem Definition
Top $k$ motif discovery

Given a multidimensional time series $T$ of length $n$ and dimensionality $D$, a window size $w$, a number of motifs $k$, and motif dimensionality $d$, the problem is to find the top $k$ motifs of dimensionality $d$ in the time series.


- We have to find the <span v-mark.blue = "1">subset of dimensions</span> that spans the motif.

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

 <br>

 - It employs the **Cyclic Convolution Theorem** to compute in $O(n\log n)$ time the distances

---
transition: fade
---

# How do we find motifs exactly?
The **Matrix Profile** Approach

 - The Matrix Profile is a data structure that allows us to find motifs in time series data efficiently.
  
 <br>

 - It employs the **Cyclic Convolution Theorem** to compute in $O(n\log n)$ time the distances
  
 <br>

 - It stores in $O(n)$ space the distance of each subsequence with its nearest neighbor

---
transition: fade
---

# How do we find motifs exactly?
The **Matrix Profile** Approach

 - The Matrix Profile is a data structure that allows us to find motifs in time series data efficiently.
  
 <br>

 - It employs the **Cyclic Convolution Theorem** to compute in $O(n\log n)$ time the distances
  
 <br>

 - It stores in $O(n)$ space the distance of each subsequence with its nearest neighbor
  
 <br>

 - We can use it to find motifs of any dimensionality, anomalies, discords and many other patterns in time series data.

---
transition: fade
---

# The **Matrix Profile** Approach
A unidimensional visualization

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

# The **Matrix Profile** Approach
A multidimensional example

![](/images/mp1.png)

---
transition: fade
---

# The **Matrix Profile** Approach
A multidimensional example

![](/images/mp2.png)

---
transition: fade
---

# The **Matrix Profile** Approach
A multidimensional example

![](/images/mp3.png)

---
transition: fade
---

# The **Matrix Profile** Approach
A multidimensional example

![](/images/mp4.png)


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
transition: fade
---

# How do we find motifs approximately?
The **EMD** Approach

A meta time series is obtained by applying **Principal Component Analysis** to the multidimentional time series. 

Motifs are then discovered with the <span v-mark.circle.green="0"> Minimum Description Length </span> principle.

Pros:
- The dimensionality of the time series does not have an impact on the discovery

Cons:
-  Motifs may be masked by useless dimensions or noise
- Additional step to retrieve the dimensions that span the motif

---
transition: fade
---

# How do we find motifs approximately?
The **Random Selection** Approach

We randomly pick **combinations** of the dimensions of the time series, symbolize them and fill a collision matrix each time we find a collision.



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

<br>

- Ideally we want to use an hash function that can 
tell us what are the colliding subsequences 
and what are the dimensions where they collide.

<br>

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

<br>

- <span v-mark.green="1"> Creating an hash index </span> of the time series

<br>

- <span v-mark.green="2"> Finding the closest pairs </span> in the hash index

<br>

- <span v-mark.green="3"> Stopping once we are sure the result is _good enough_ </span>

---
transition: fade
layout: two-cols
---

# Hash Index
Discretized Random Projections

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

 - In particular, we hash each dimension of the subsequences <span v-mark.blue = "1">independently</span>.

 - We end up with a <span v-mark.blue = "2">codebook of $d$ hash values</span> for each subsequence.

---
transition: fade
---

# Closest Pairs
Weighting the collisions

Once we have built the hash index, we can scan the bins and look for colliding subsequences.

In particular we track the <span v-mark.green="1"> weight </span> of the collisions, which is defined as:
$$
    w\left(T_a, T_b\right) = \sum_{i=0}^{d-1} \left(h\left(T_a[i]\right) == h\left(T_b[i]\right)\right)
$$


where $h(T_a[i])$ is the hash value of the $i$-th dimension of the subsequence $T_a$ and $==$ is a binary operator.
<br>



---
transition: fade
---

# Closest Pairs
Weighting the collisions

Once we have built the hash index, we can scan the bins and look for colliding subsequences.

In particular we track the <span v-mark.green="0"> weight </span> of the collisions, which is defined as:
$$
    w\left(T_a, T_b\right) = \sum_{i=0}^{d-1} \left(h\left(T_a[i]\right) == h\left(T_b[i]\right)\right)
$$


where $h(T_a[i])$ is the hash value of the $i$-th dimension of the subsequence $T_a$ and $==$ is a binary operator.
<br>

We then compute the distance of the pairs of <span v-mark.green="0"> subsequences whose weight is $>d$ </span>, the requested motif dimensionality, and track our top-$k$ motifs in a priority queue.
---
transition: fade
---

# A Stopping Criterion
using LSH properties

- Let $p(dist)$ be the probability of sharing the same hash value at distance $dist$, from now we assume $dist$ is implicit.
<br>

---
transition: fade
---

# A Stopping Criterion
using LSH properties

- Let $p(dist)$ be the probability of sharing the same hash value at distance $dist$, from now we assume $dist$ is implicit.
<br>

- Now, let $i$ be the number of the current concatenation and $j$ be the number of the hash repetition.
<br>
Then, the probability of sharing the same hash value is:
$$
  \left(1-p^i\right)^j
$$

---
transition: fade
---

# A Stopping Criterion
using LSH properties

- Let $p(dist)$ be the probability of sharing the same hash value at distance $dist$, from now we assume $dist$ is implicit.
<br>

- Now, let $i$ be the number of the current concatenation and $j$ be the number of the hash repetition.
<br>
Then, the probability of sharing the same hash value is:
$$
  \left(1-p^i\right)^j
$$

- Using the approach of PUFFINN*, we can define the probability of sharing the same hash value as:
  $$
        \left\{
          \begin{aligned}
          &\left(1-p^i\right)^j \quad &\textrm{ if }~~ i=K \\
          &\left(1-p^i\right)^j \cdot \left(1-p^{i+1}\right)^{L-j} \quad&\textrm{ otherwise}
          \end{aligned}
        \right.
  $$
  with $K$ the number of hash repetitions and $L$ the number of concatenations, respectively.


###### *([Aumüller et al., 2020](https://arxiv.org/abs/1906.12211))
---
transition: fade
---

# A Running Example
given time series $T$, window $w=10$, number of motifs to find $k=3$ and motif dimensionality $d=2$

<LeitMotifs />

---
transition: fade
---

# Experimental Results
The datasets

<table class="w-full text-xs border-collapse border border-gray-300">
  <thead>
        <th class="border px-1 py-0.5">dataset</th>
        <th class="border px-1 py-0.5">n (length)</th>
        <th class="border px-1 py-0.5">D (dimensionality)</th>
        <th class="border px-1 py-0.5">w (window)</th>
        <th class="border px-1 py-0.5">d (motif dimensionality)</th>
  </thead>
    <tbody>
    <tr>
      <td class="border px-1 py-0.5">potentials</td>
      <td class="border px-1 py-0.5">2 500</td>
      <td class="border px-1 py-0.5">8</td>
      <td class="border px-1 py-0.5">50</td>
      <td class="border px-1 py-0.5">8</td>
    </tr>
    <tr>
      <td class="border px-1 py-0.5">evaporator</td>
      <td class="border px-1 py-0.5">7 000</td>
      <td class="border px-1 py-0.5">5</td>
      <td class="border px-1 py-0.5">75</td>
      <td class="border px-1 py-0.5">2</td>
    </tr>
    <tr>
      <td class="border px-1 py-0.5">RUTH</td>
      <td class="border px-1 py-0.5">14 859</td>
      <td class="border px-1 py-0.5">32</td>
      <td class="border px-1 py-0.5">500</td>
      <td class="border px-1 py-0.5">8</td>
    </tr>    
    <tr>
      <td class="border px-1 py-0.5">weather</td>
      <td class="border px-1 py-0.5">100 057</td>
      <td class="border px-1 py-0.5">8</td>
      <td class="border px-1 py-0.5">5000</td>
      <td class="border px-1 py-0.5">2</td>
    </tr>    
    <tr>
      <td class="border px-1 py-0.5">whales</td>
      <td class="border px-1 py-0.5">450 001</td>
      <td class="border px-1 py-0.5">32</td>
      <td class="border px-1 py-0.5">300</td>
      <td class="border px-1 py-0.5">6</td>
    </tr>    
    <tr>
      <td class="border px-1 py-0.5">quake</td>
      <td class="border px-1 py-0.5">6 440 998</td>
      <td class="border px-1 py-0.5">32</td>
      <td class="border px-1 py-0.5">100</td>
      <td class="border px-1 py-0.5">4</td>
    </tr>    
    <tr>
      <td class="border px-1 py-0.5">el_load</td>
      <td class="border px-1 py-0.5">6 960 008</td>
      <td class="border px-1 py-0.5">10</td>
      <td class="border px-1 py-0.5">1000</td>
      <td class="border px-1 py-0.5">5</td>
    </tr>    
    <tr>
      <td class="border px-1 py-0.5">LTMM</td>
      <td class="border px-1 py-0.5">25 132 289</td>
      <td class="border px-1 py-0.5">6</td>
      <td class="border px-1 py-0.5">200</td>
      <td class="border px-1 py-0.5">3</td>
    </tr>
    </tbody>
</table>

---
transition: fade
---

# Experimental Results
Time to find the top motif in seconds

<table class="w-full text-xs border-collapse border border-gray-300">
  <thead>
    <tr>
      <th class="border px-1 py-0.5">dataset</th>
      <th class="border px-1 py-0.5" colspan="2">LEIT-motifs</th>
      <th class="border px-1 py-0.5">MSTUMP</th>
      <th class="border px-1 py-0.5">EMD</th>
      <th class="border px-1 py-0.5">RP</th>
    </tr>
    <tr>
      <th></th>
      <th class="border px-1 py-0.5">Index build</th>
      <th class="border px-1 py-0.5">Total</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="border px-1 py-0.5">potentials</td>
      <td class="border px-1 py-0.5">0.11</td>
      <td class="border px-1 py-0.5 bg-green-100 text-black">0.51</td>
      <td class="border px-1 py-0.5">3.65</td>
      <td class="border px-1 py-0.5">4.80</td>
      <td class="border px-1 py-0.5">3.20</td>
    </tr>
    <tr>
      <td class="border px-1 py-0.5">evaporator</td>
      <td class="border px-1 py-0.5">0.16</td>
      <td class="border px-1 py-0.5 bg-green-100 text-black">0.55</td>
      <td class="border px-1 py-0.5">4.45</td>
      <td class="border px-1 py-0.5">12.95</td>
      <td class="border px-1 py-0.5">6.78</td>
    </tr>
    <tr>
      <td class="border px-1 py-0.5">RUTH</td>
      <td class="border px-1 py-0.5">2.91</td>
      <td class="border px-1 py-0.5 bg-green-100 text-black">8.10</td>
      <td class="border px-1 py-0.5">84.04</td>
      <td class="border px-1 py-0.5">1.5h</td>
      <td class="border px-1 py-0.5">2.3h</td>
    </tr>
    <tr>
      <td class="border px-1 py-0.5">weather</td>
      <td class="border px-1 py-0.5">15.04</td>
      <td class="border px-1 py-0.5 bg-green-100 text-black">33.37</td>
      <td class="border px-1 py-0.5">1035.73</td>
      <td class="border px-1 py-0.5">-</td>
      <td class="border px-1 py-0.5">1.2h</td>
    </tr>
    <tr>
      <td class="border px-1 py-0.5">whales</td>
      <td class="border px-1 py-0.5">60.67</td>
      <td class="border px-1 py-0.5 bg-green-100 text-black">2.2 h</td>
      <td class="border px-1 py-0.5">(2.7 days)</td>
      <td class="border px-1 py-0.5">-</td>
      <td class="border px-1 py-0.5">-</td>
    </tr>
    <tr>
      <td class="border px-1 py-0.5">quake</td>
      <td class="border px-1 py-0.5">175.3</td>
      <td class="border px-1 py-0.5 bg-green-100 text-black">3.6 h</td>
      <td class="border px-1 py-0.5">(7.2 days)</td>
      <td class="border px-1 py-0.5">-</td>
      <td class="border px-1 py-0.5">-</td>
    </tr>
    <tr>
      <td class="border px-1 py-0.5">el_load</td>
      <td class="border px-1 py-0.5">180.2</td>
      <td class="border px-1 py-0.5 bg-green-100 text-black">2.8 h</td>
      <td class="border px-1 py-0.5">(8.4 days)</td>
      <td class="border px-1 py-0.5">-</td>
      <td class="border px-1 py-0.5">-</td>
    </tr>
    <tr>
      <td class="border px-1 py-0.5">LTMM</td>
      <td class="border px-1 py-0.5">240.6</td>
      <td class="border px-1 py-0.5 bg-green-100 text-black">15.6 h</td>
      <td class="border px-1 py-0.5">(11.8 days)</td>
      <td class="border px-1 py-0.5">-</td>
      <td class="border px-1 py-0.5">-</td>
    </tr>
  </tbody>
</table>


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
Can we also find motifs of different dimensionalities like the Matrix Profile?

- Yes, we can use our hash index to find motifs of any dimensionality


---
transition: fade
---

# Finding motifs of different dimensionalities
Can we also find motifs of different dimensionalities like the Matrix Profile?

- Yes, we can use our hash index to find motifs of any dimensionality
- What happens to our collision weight?

  We start by comparing pairs whose weight $w\geq 2$, and track our top-$k$ motifs in a set of $D$ priority queues, once the motif of dimensionality $d=2$ is confirmed we compare only the pairs whose $w\geq 3$, and so on up to $D$.

---
transition: fade
layout: two-cols
---

# Finding motifs of different dimensionalities
Can we also find motifs of different dimensionalities like the Matrix Profile?
::right::

<div style="text-align: right;">
  <img src="/images/multisub_small-1.png" style="width:400px;" />
</div>

---
transition: fade
---

# Complexity Analysis
How many distances we compute?

On a $D$-dimensional time series $T$ of length $n$, with $k\geq1$, $d\in [1,D]$, $\delta\in (0,1)$ we compute
$$
  O\left(
    n^{1 + 1/c} \log\frac{1}{\delta} + Lk
  \right)
$$
distances, where $c$ is the <span v-mark.orange="1"> contrast </span> of the time series and $L$ the hash repetitions.

---
transition: fade
---

# Complexity Analysis
How many distances we compute?

On a $D$-dimensional time series $T$ of length $n$, with $k\geq1$, $d\in [1,D]$, $\delta\in (0,1)$ we compute
$$
  O\left(
    n^{1 + 1/c} \log\frac{1}{\delta} + Lk
  \right)
$$
distances, where $c$ is the <span v-mark.orange="0"> contrast </span> of the time series and $L$ the hash repetitions.

Intuition:
 -  If a motif is very different from the rest of the time series, then the hash index will easily isolate it, we have an <span v-mark.green="1">high contrast </span>.
 - If the time series is very noisy and there are several similar patterns, then we have a <span v-mark.red="2">low contrast </span>.
 - $c\geq 1$, if $c=1$ then all the subsequences are equal.

---
transition: fade
layout: center
---

# <span class="leit-title">ThanKs foR <br> youR AttentioN</span>

<TSBackground />
