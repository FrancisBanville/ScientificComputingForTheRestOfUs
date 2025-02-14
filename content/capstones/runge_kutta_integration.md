---
title: Runge-Kutta integration
slug: runge_kutta_integration
weight: 2
packages:
  - StatsPlots
concepts:
  - writing functions
  - numerical precision
  - arrays
  - keyword arguments
contributors:
  - tpoisot
---

Numerical integration, the search for solutions of differential equations, is a
hallmark of scientific computing. In this lesson, we will see how we can apply
multipe concepts to write our own routine for the second-order Runge-Kutta
method. In practice, it is *never* recommended to write one's own routine for
numerical integration, as there are specific packages to handle this task. In
Julia, this is
[`DifferentialEquations.jl`](http://docs.juliadiffeq.org/latest/). This being
said, writing a Runge-Kutta method is an interesting exercise.

Glossing over a few details, numerical integration requires a function $f$,
giving the derivative at time $t$, a timestep $h$ at which to evaluate $f$, and
an initial quantity $y$. There are multiple notations, but for this lesson we
will adopt $y(t)$ for the value of $y$ at time $t$. The second-order Runge-Kutta
method (RK2) works by evaluating the derivative at time $t+h/2$, and using this
information to correct the initial assessment. This helps (very moderately) in
problems where $y$ varies a lot over short periods of time.

In practice, RK2 is described by the following system of equations:

$$y(t+\frac{h}{2}) = y(t)+\frac{h}{2}f\left(t, y(t)\right)$$

$$y'(t+\frac{h}{2}) = f\left(t+\frac{h}{2}, y(t+\frac{h}{2})\right)$$

$$y(t+h) = y(t) + h y'(t+\frac{h}{2})$$

But before we start writing this in code, let's think about the function we want
to integrate for a minute or two. In this example, we will use the situation of
two species, one of which ($x$) is a prey, the other ($y$) a predator. The equations we use
to model this system are:

$$\frac{x'}{x} = r - γ y$$
$$\frac{y'}{y} = \beta x - δ$$

In other words, the prey grows at rate $r$. Every unit of predator removes
$\gamma$ units of prey, which are converted into $\beta$ unit of predators;
finally, predators decay at rate $\delta$.

Most of these parameters are things we might want to change, so our function
should be able to accept them as parameters. We may also want to substitute
another problem later, instead of working on the competition model. For this
reason, it is better to write a fonction which is as general as possible.

We know that all of our problems (*i.e.* all of our models) will have two things
in common: they will require the time $t$, and the value $u$; the rest can go in
keyword arguments. This gives us a template to write the function for the
competitive model:

````julia
function predation(t::Float64, u::Vector{Float64}; r=1.0, γ=1.0, β=1.0, δ=1.0)
  x, y = u
  dx = x*r-γ*x*y
  dy = β*x*y-δ*y
  return [dx, dy]
end
````


````
predation (generic function with 1 method)
````





There are *many* things that can go wrong with this function -- so we may want
to add a few checks before we call it. This is an interesting exercise to do,
and you can re-read through the lesson on [avoiding mistakes]({{< ref
"/lessons/avoiding_mistakes.md" >}}) to refresh your memory. For now, we will
use this basic (but unsafe) version.

Let's try! If we have a single species (the preys), and we give it any initial
population larger than 0, then it should grow (and the population of the
predator should not change):

````julia
predation(0.0, [0.1, 0.0])
````


````
2-element Array{Float64,1}:
 0.1
 0.0
````





Nothing stops us from running this function a few times, to figure out if it
gives the correct dynamics:

````julia
u0 = [0.1, 0.1]
population = [u0]
for i in 2:30
  push!(population, population[i-1] + predation(0.0, population[i-1]))
end
````





We can plot this:

````julia
using StatsPlots
plot(hcat(population...)[1,:], c=:teal, lab="Prey",
  xlim=(0,30), ylim=(0,5), frame=:origin,
  xlab="Time", ylab="Abundance"
  )
plot!(hcat(population...)[2,:], c=:purple, lab="Predator")
````


{{< figure src="../figures/runge_kutta_integration_4_1.png" title="Even with a simple model, our naive integration scheme causes the population to crash. This is why we need to use a more robust integration method!"  >}}


We obviously need to start thinking about the RK2 integrator! We want our RK2
integrator (a function named `rk2`) to return two values: an array of times,
going from $t_0$ to $t_f$ by steps of $h$, and an array of population sizes at
every point. This means that we can start thinking about the arguments:
`t=(t0,tf)` is a tuple with the beginning and end time; `h` is the integration
step. None of these have obvious defaults, but we will still put them as
default, keyword arguments, to make sure that we can run our function with a
minimum number of keystrokes. If this seems contradictory to the best practices
we mentionned in some lessons, it's because it really is! But best practices are
not laws, and sometimes it is worth applying critical judgement.

Our function will therefore look something like

```raw
function rk2(;t=0.0=>100.0, h=0.1)
  # something
  # something else
  T = collect(t.first:h:t.second)
  return something, T
end
```

The notation we use for `t` is called a `Pair` (and is covered in the Julia
documentation). The next step is to think about the "meat" of the function: for
every timestep, we want to apply the RK2 equations that are given early in this
lesson. Assuming that the current population is stored at position `i` of an
object called `U`, this would look like:

```raw
y_th2 = U[i] .+ 0.5*h .* f(ti, U[i]; p...)
yprime_th2 = f(ti+0.5*h, y_th2; p...)
y_th = U[i] .+ h*yprime_th2
U[i+1] = y_th
```

There are a few things worth looking at here. First, note that we use the `.`
notation, to make sure that operations are vectorized. Second, we call the `f`
function (which here would be our `predation` function) using an
argument called `p...`. This syntax allows a function to *capture keyword
arguments, and pass them to another function. Let's have a look:

````julia
function a(x; y=2.0, z=3.0)
  return x + y + z
end
function b(x; k...)
  return a(x; k...)
end

b(1.0) # Should be 6
````


````
6.0
````



````julia
b(1.0; y=0.0) # Should be 4
````


````
4.0
````





In short, this allows passing "all the keyword arguments" from a function to
another. This is really useful in our context, because although we currently use
the competition model, we may want to change it later, possibly to something
that accepts different keyword arguments! This gives us the two extra parameters
we need to add to `rk2`: the function `f`, and the term to catch arguments `p...`.

```raw
function rk2(f; t=0.0=>100.0, h=0.1, p...)
  # something
  for some things
    y_th2 = U[i] .+ 0.5*h .* f(ti, U[i]; p...)
    yprime_th2 = f(ti+0.5*h, y_th2; p...)
    y_th = U[i] .+ h*yprime_th2
    U[i+1] = y_th
  end
  # something else
  T = collect(t.first:h:t.second)
  return U T
end
```

Believe it or not, this is almost finished! All we need is to set a value for
`U`, and to iterate over all values of `T`:

````julia
function rk2(u0, f; t=0.0=>100.0, h=0.1, p...)
  T = collect(t.first:h:t.second)
  U = typeof(u0)[]
  push!(U, u0)
  for (i,ti) in enumerate(T[2:end])
    y_th2 = U[i] + 0.5*h * f(ti, U[i]; p...)
    yprime_th2 = f(ti+0.5*h, y_th2; p...)
    y_th = U[i] + h*yprime_th2
    push!(U, y_th)
  end
  return (T, hcat(U...)')
end
````


````
rk2 (generic function with 1 method)
````





Let's try to run the same example as before:

````julia
t, u = rk2([0.1, 0.2], predation; t=1.0=>30.0, r=0.9, h=0.05)
plot(t, u[:,1], c=:teal, lab="Prey",
  xlim=(0,30), ylim=(0,5), frame=:origin,
  xlab="Time", ylab="Abundance"
  )
plot!(t, u[:,2], c=:purple, lab="Predator")
````


{{< figure src="../figures/runge_kutta_integration_8_1.png" title="Using a second-order Runge-Kutta method, we can get the correct result. This predation model is not very complex, but it illustrates really well the importance of taking numerical integration seriously!"  >}}


Notice that we had previously not used $t$ in the integration -- but because our
function is general, we can add something like a seasonal effect on the
intensity of predation -- specifically, the effect of the predator on the prey
is a sinusoidal function of time, with "high" and "low" seasons.

````julia
function seasonal_predation(t::Float64, u::Vector{Float64}; r=1.0, γ=1.0, β=1.0, δ=1.0)
  x, y = u
  dx = x*r-(γ+sin(2.0*t)*0.15)*x*y
  dy = β*x*y-δ*y
  return [dx, dy]
end
````


````
seasonal_predation (generic function with 1 method)
````





This require no changes to the code to produce the figure:

````julia
t, u = rk2([0.1, 0.2], seasonal_predation; t=1.0=>200.0, r=0.9, h=1e-2)
plot(t, u[:,1], c=:teal, lab="Prey",
  xlim=(0,50), ylim=(0,5), frame=:origin,
  xlab="Time", ylab="Abundance"
  )
plot!(t, u[:,2], c=:purple, lab="Predator")
````


{{< figure src="../figures/runge_kutta_integration_10_1.png" title="Because we have written a general function, generating a new result is as simple as changing the name of the function we want to integrate (and adding the correct keyword arguments, if needed)!"  >}}