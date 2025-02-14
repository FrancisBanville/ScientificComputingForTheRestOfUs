---
title: Avoiding mistakes
weight: 4
status: construction
concepts:
  - defensive programming
  - writing functions
  - assertion
  - unit testing
packages:
  - Test
  - Statistics
contributors:
  - tpoisot
---

## We can't avoid mistakes

But we can work as cautiously as possible, to make sure we catch them in time.
It is always better to try and fail to run something, than to have the operation
keep going and accumulating mistakes.

There are four types of mistakes to look out for: mistakes in the code,
confusing interface, issues with arguments, and lack of integration. Some are
caused by the programmer, and some are caused by the user. But in the context of
writing code for science, the programmer and the user are often the same person,
and so passing the blame around ends up being a very frustrating exercise. Even
if it were not the case, user mistakes can come from sub-optimal design. It is
crucial to work in a way that protects everyone against mistakes.

{{% callout opinion %}} One of our golden rules is "fail early, fail often, and
fail explicitely" -- it is frustrating to have to restart an analysis, but this
is preferable to running an analysis that keeps accumulating issues we may or
may not detect! {{% /callout %}}


In this lesson, you may note that we will switch perspective frequently, from
user to developper. This is because, in our own experience, this is a fair
representation of the way we work. We try to write something (developper), then
apply it to a specific problem (user), then figure out there is an issue and
switch back to developper mode.

## After this lesson, you will be able to ...

- ... use defensive programming
- ... write basic tests to ensure that the program fails when it should
- ... think about function design in a way that minimizes confusion

But let's do something a little bit different. Instead of writing a function, we
will start by thinking about its behavior. In this example, we want to write a
function that will calculate an average. That's it. Specifically, it will
calculate the singal-to-noise ration in an array of numbers, using the S = μ/σ
expression, where μ is the average and σ the standard deviation.

What we will do is *declare* the function, but put nothing in it -- in each of
the sections, we will add a few lines to the function, to make it *work*. For
now, we want a function that does *nothing*.

````julia
function snr(x::Vector{T}) where {T <: Number}
end
````


````
snr (generic function with 1 method)
````





## Using the wrong arguments

The first thing that can go wrong with this function is calling it with the
wrong arguments. In a sense, we have limited this risk because we took advantage
of Julia's type annotation system, and so we can *only* call our function when
the argument is an array of numbers.

But with the definition of signal-to-noise ratio we picked, it only makes sense
to apply this function when all elements of `x` are *non-negative*. So this is
the first thing our function should check. But instead of changing the function,
we will first *test* its behavior.

{{% callout information %}}
This process, starting by the tests and writing the function after, is called
*test-driven development*. It is not the only way to proceed, but we think it is
an interesting practice to experiment with.
{{% /callout %}}

Let's start by documenting our "normal" behavior: calling the function with only
positive or null numbers should not give any warning or error message:

````julia
using Test
@test_nowarn snr([0.0, 1.0, 2.0])
````





This test is *passing*: running the function with this argument gives no warning
(because the function currently does nothing, but that is besides the point).
And we also want our function to return an error when we call it with negative
arguments:

````julia
@test_throws DomainError snr([-1.0, 1.0, 2.0])
````


````
Test Failed at none:1
  Expression: snr([-1.0, 1.0, 2.0])
    Expected: DomainError
  No exception thrown
````


<pre class="julia-error">
ERROR: Test.FallbackTestSetException&#40;&quot;There was an error during testing&quot;&#41;
</pre>




Well, this one, predictabily, is failing! So here is our first step: we need to
add something to our function to make it return a `DomainError` -- this is a
way to tell our user that something is not quite right with the arguments.

{{% callout information %}}
Julia, like most modern languages, has a well developed system for throwing
errors, and when you write code that becomes a little bit complex, or is meant
to be used by others, it is worth spending some time reading it.
{{% /callout %}}

We can add a line checking the sign of the smallest value of `x` -- if this is
lower than 0 (and specifically, of the `zero` value of the type `T` of the
elements), then we *throw* an error.

````julia
function snr(x::Vector{T}) where {T <: Number}
  minimum(x) .< zero(T) && throw(DomainError(minimum(x), "all values passed must be positive or null."))
end
````


````
snr (generic function with 1 method)
````





Note that our error is *informative*: it gives a message explaining what went
wrong. Now that we have added this line, we need to make sure that *both* our
tests pass!

````julia
@test_nowarn snr([0.0, 1.0, 2.0])
@test_throws DomainError snr([-1.0, 1.0, 2.0])
````


````
Test Passed
      Thrown: DomainError
````





Yeah! Now, because our function involves calculating a standard deviation, we
may want to restrict its application to inputs with more than 3 elements. This
is also something we can test, and throw an error:

````julia
function snr(x::Vector{T}) where {T <: Number}
  length(x) < 3 && throw(ArgumentError("A minimum of three values must be provided."))
  minimum(x) .< zero(T) && throw(DomainError(minimum(x), "all values passed must be positive or null."))
end
````


````
snr (generic function with 1 method)
````





And now, let's run *all of the previous tests*, but also a new one.

````julia
@test_nowarn snr([0.0, 1.0, 2.0])
@test_throws DomainError snr([-1.0, 1.0, 2.0])
@test_throws ArgumentError snr([1.0, 1.0])
````


````
Test Passed
      Thrown: ArgumentError
````





At this point, our function still does nothing. In fact, it does *less* than
nothing, since it will refuse to run in some situations. This practice is
generally refered to as *defensive programming*: we want to perform the actual
computation only when we are confident that the conditions to run it are met.

For now, we have done enough work. Let's work on the code later, and we will
instead improve the "interface" of our function.

## Confusing interface

{{% callout opinion %}}
This entire section is a matter of opinions -- we encourage you to have a look
at your favorite language's manual of style. [Julia has
one](https://docs.julialang.org/en/v1/manual/style-guide/index.html) which we
think is good, and we generally follow, except when we don't.
{{% /callout %}}

As it is written, our function does not have a particularly explicit name. Maybe
`snr` makes sense to us because it is fresh in our mind; but will it still be
true in a week? A month? Let's instead aim for something more explicit:
`signaltonoiseratio`.

````julia
function signaltonoiseratio(x::Vector{T}) where {T <: Number}
  length(x) < 3 && throw(ArgumentError("A minimum of three values must be provided."))
  minimum(x) .< zero(T) && throw(DomainError(minimum(x), "all values passed must be positive or null."))
end
````


````
signaltonoiseratio (generic function with 1 method)
````





It is longer, but it is also more explicit. And in most cases, typing `sign` and
pressing `Tab` will autocomplete to the full function name, so there is minimal
effort involved.

Another source of mistakes is to have non-descriptive names for variables. In
the existing function, the most important variable is called `x`: this means
nothing. We will replace `x` by something more informative, such as
`measurement`:

````julia
function signaltonoiseratio(measurement::Vector{T}) where {T <: Number}
  length(measurement) < 3 && throw(ArgumentError("A minimum of three values must be provided."))
  minimum(measurement) .< zero(T) && throw(DomainError(minimum(measurement), "all values passed must be positive or null."))
end
````


````
signaltonoiseratio (generic function with 1 method)
````





At this point, our function still does *absolutely nothing*, but we can be sure
that it is named in a way that makes it easier to reason about, will refuse to
run when the arguments are wrong, and uses variable names that mean something.
In short, it does nothing, but it does it really well.

## Mistakes in the code


## Lack of integration

- mistake in functions
- integration issues
