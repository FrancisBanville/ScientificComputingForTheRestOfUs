---
title: Parallel computing
slug: parallelcomputing
layout: page
status: construction
draft: true
concepts:
  - optimization
  - parallel computing
packages:
  - Distributed
contributors:
  - tpoisot
weight: 4
---

Sometimes, we really need more than one CPU to get things done in a reasonable
amount of time.

````julia
using Distributed
````



````julia
addprocs(2)
````


````
2-element Array{Int64,1}:
 6
 7
````


