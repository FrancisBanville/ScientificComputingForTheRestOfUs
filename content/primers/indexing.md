---
title: Indexing and arrays dimensions
slug: indexing
concepts:
  - arrays
  - indexing
  - slices
  - sequences
contributors:
  - tpoisot
status: construction
draft: true
weight: 1
---

The overwhelming majority of data we will need to manipulate will be stored in
vectors, or matrices, or other types of multi-dimensional structures. Being able
to locate and modify information stored in these structure is one of the most
important skills you can develop. *Julia* has a number of ways to facilitate and
refine this, and the point of this primer is to showcase the most useful of
them.

````julia
x = [i for i in 1:5]
````


````
5-element Array{Int64,1}:
 1
 2
 3
 4
 5
````


