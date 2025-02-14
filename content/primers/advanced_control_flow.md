---
title: Advanced control flow
slug: advanced_control_flow
concepts:
  - control flow
contributors:
  - tpoisot
weight: 1
---

In the [first lesson]({{< ref "/lessons/control_flow.md">}}), we have
presented the flow of execution as something largely unescapable. In practice,
there are two important operations we may want to perform: skip an iteration, or
stop the iteration process entirely. In this primer, we will see how this can be
achieved using the `break` and `next` keywords.

Understanding how `next` and `break` can be used is extremely useful as you
start developing more complex workflows. It can save a lot of time when you know
that you want to avoid some operations, and is therefore worth understanding.

Let's imagine that we have the following loop:

```raw
i = 1
while true
  println(i)
  global i += 1
end
```

You should **not** run this, because it will run forever. This loop as no
*termination condition*, and therefore will go on forever. Let's say we want to
stop it after `i >= 5` -- this is something we can do using the `break` keyword:

````julia
i = 1
while true
  println(i)
  i < 5 || break
  global i += 1
end
````


````
1
2
3
4
5
````





The line we added (`i < 5 || break`) will check that `i` is less than or equal
to 5 -- if this is true, then it will `break` the iteration, *i.e.* stop. Note
that `i < 5 || break` uses [short-circuit
evaluation](https://docs.julialang.org/en/v1.0/manual/control-flow/#Short-Circuit-Evaluation-1),
which is a wonderful bit of notation: simple yet efficient.

The `continue` keyword has an extremely useful role too -- it allows you to skip
one iteration. Let's say that we want to perform an operation only on number
smaller than 1/3:

````julia
for r in rand(10)
  r ≥ 1/3 || continue
  println(round(r; digits=2))
end
````


````
1.0
0.62
0.84
0.46
0.99
````





In short, `continue` will *skip ahead* to the next element in the iteration.
This can be very important to avoid performing operations on objects that are
not relevant.
