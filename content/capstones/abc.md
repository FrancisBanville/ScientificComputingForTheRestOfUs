---
title: Approximate Bayesian Computation
weight: 1
packages:
  - StatsPlots
  - StatsBase
  - Statistics
  - Distributions
concepts:
  - arrays
  - control flow
contributors:
  - tpoisot
---


[Approximate Bayesian
computation](https://en.wikipedia.org/wiki/Approximate_Bayesian_computation), or
ABC for short, is a very useful heuristic to estimate the posterior distribution
of model parameters, specifically when the analytical expression of the
likelihood function is unavailable (or when we can't be bothered to figure it
out). The theory on how ABC works will not be covered here in detail, so reading
the previous article is highly recommended.

We will rely on a few packages for this example:

````julia
using StatsPlots
using Statistics
using StatsBase
using Distributions
````





Let us now imagine an island. It is a small island, no more than a few meters in
diameter, in the Florida keys. Every year, for 20 years, a few biologists comb
through the island, to figure out whether or not *Eurycotis floridana* is
present or absent. This results in a timeseries, like this:

````
Year 1	absent
Year 2	absent
Year 3	present
Year 4	present
Year 5	absent
Year 6	absent
Year 7	present
Year 8	absent
Year 9	present
Year 10	absent
````





To begin with, let us see how we can model the presence/absence of this species.
We will make two assumptions. First, the biological process here can be
represented by modeling the probabilities of *transition*, *i.e.* the chance of
changing from one state to another. Second, there is a small chance of *error*
when measuring the state of the system. Specifically, while detecting at least
one individual means that there is no chance that the species is absent, *not*
detecting any individual can also mean that there were present in low density,
and have not been detected.

We can represent this system using the following figure:

{{< mermaid >}}
graph LR

subgraph True state
present -- e --> absent
absent -- c --> present
absent -- 1-c --> absent
present -- 1-e --> present
end

subgraph Measured state
absent --> InsectAbsent[absent]
present -- 1-m --> InsectPresent[present]
present -- m --> InsectAbsent[absent]
end
{{< /mermaid >}}

In the *true state* box, the rates represent the probabilities of state change.
The rates between *true state* and *measured state* represent the probabilities
of making the wrong measurement. We now have enough to write a simple function
to simulate this model:

````julia
function island(e::T, c::T, m::T; n=200) where {T <: AbstractFloat}
  @assert 0.0 ≤ e ≤ 1.0
  @assert 0.0 ≤ c ≤ 1.0
  @assert 0.0 ≤ m ≤ 1.0
  true_state = zeros(Bool, n)
  measured_state = zeros(Bool, n)
  measured_state[1] = true_state[1]
  for year in 2:n
    true_state[year] = true_state[year-1] ? rand() ≥ e : rand() < c
    measured_state[year] = true_state[year] ? rand() ≥ m : false
  end
  return measured_state
end
````


````
island (generic function with 1 method)
````





In ABC, one key notion is the idea of "summary statistics", *i.e.* the act of
compressing the empirical data and model output to something that can be
meaningfully compared. Here, we will work with two informations, namely the rate
of transition, and the temporal occupancy. In ABC, the summary statistics are
*key* in determining the validity of the results. Using too much exposes you to
the curse of dimensionality, and using too few (or the wrong ones) can lead to
spurious conclusions. Another few indicators we could have used are apparent
colonisation events (when the island appeared empty one year, then occupied the
next), apparent extinction events (when the island appears occupied one year,
and empty the next), and finally the number of times where the sequence
occupied/empty/occupied appears. Feel free to try and adapt the code after you
have been through this lesson once to use these and other indicators, and see
how the results change.

In any case, here is our function to summarize a sequence of observations:

````julia
function summary(t::Vector{Bool})
  transitions = 0.0
  for i in 2:length(t)
    if t[i] ≠ t[i-1]
      transitions = transitions+1.0
    end
  end
  occupancy = sum(t)/length(t)
  return [occupancy, transitions/(length(t)-1)]
end
````


````
summary (generic function with 1 method)
````





Do you like it? It's OK I guess, but I'm not a big fan. As we discussed in the
previous paragraph, we may want to use different summary statistics, and this
would require re-writing this function. This is a good example of our code being
insufficiently modular: each summary statistic should be its own function. Let's
correct this.

````julia
occupancy(t::Vector{Bool}) = sum(t)/length(t)

function transitions(t::Vector{Bool})
   n = 0.0
   for i in 2:length(t)
      if t[i] ≠ t[i-1]
         n = n+1.0
      end
   end
   return n/(length(t)-1)
end
````


````
transitions (generic function with 1 method)
````





We have defined two functions, each of which returns the value of one indicator.
We can now write a declaration for `summary` that will accept the states
timeseries, *and* an array of functions to measure the indicators:

````julia
function summary(t::Vector{Bool}, f::Vector{T}) where {T <: Function}
   s = zeros(Float64, length(f))
   for (i, fn) in enumerate(f)
      s[i] = fn(t)
   end
   return s
end
````


````
summary (generic function with 2 methods)
````





Is it perfect? No. Is it an improvement over the previous implementation?
Assuredly, so let's roll with this. We can apply this function to a simulation
with parameters $(e=0.2, c=0.6, m=0.1)$, and get the summary statistics:

````julia
summary(
   island(0.2, 0.6, 0.1),   # simulation output
   [occupancy, transitions] # list of things to measure
   )
````


````
2-element Array{Float64,1}:
 0.64               
 0.38190954773869346
````





At this point, we need empirical data to feed the model. Let's say that over
twenty years, the species has been observed from year 4 to 12, then 14 to 15,
and finally from year 17 to 20. We can write this as:

````julia
empirical_data = zeros(Bool, 20)
empirical_data[4:12] .= true
empirical_data[14:15] .= true
empirical_data[17:20] .= true
````





Your biologist colleague would be very happy if you could get to an estimate of
the parameters that govern the presence of this insect on the island. This is an
easy enough task to do with ABC.

We can measure the statistics of this timeseries:

````julia
summary(empirical_data)
````


````
2-element Array{Float64,1}:
 0.75              
 0.2631578947368421
````





And just like this, we are ready to start the ABC process. We will start by
deciding on priors for all three parameters. We could model them in a variety of
ways, including β laws, and truncated distributions. Let's go with the later, as
it is a neat illustration of the `Distributions` package. We picked a value of
colonisation higher than extinction (as the species seems to persist), and a
relatively low rate of false absences.

````julia
Dc = Truncated(Normal(0.6, 0.1), 0.0, 1.0)
De = Truncated(Normal(0.3, 0.1), 0.0, 1.0)
Dm = Truncated(Normal(0.1, 0.3), 0.0, 1.0)
````





In this algorithm (as with any other Bayesian application), priors are
*determinant* in the success of the process; indeed, the posterior distribution
will be a subset of the prior, because (as we discuss below) ABC relies on
*rejection* sampling, *i.e.* it winnows parameter values that make little sense
from the original distribution. Picking a prior which is too narrow will prevent
the algorithm from converging onto the correct posterior. Conversely, picking a
prior that is too flat (let's say a uniform distribution that goes from negative
infinity to positive infinity) means that the time to convergence will explode.

We will generate 10⁶ samples using our simulation model, and get the summary
statistics for all of them. Note that the simulations run for 200 timesteps, as
opposed to 20 for the empirical data. This is perfectly fine, because ABC works
on summary statistics. An interesting exercise is to change the number of
timesteps in the simulations, and see if/how the posterior distributions react.

````julia
N = 1_000_000
Sc = rand(Dc, N)
Se = rand(De, N)
Sm = rand(Dm, N)
simulated_results = summary.(map(i -> island(Se[i], Sc[i], Sm[i]), 1:N))
````





{{< callout opinion >}}
ABC is an excellent exercise for parallel computing! Because the model runs are
independant, this is an "embarrassingly parallel" problem. After reading the
[parallel computing](/primers/parallelcomputing/) primer, you may want to try it
out on this problem.
{{< /callout >}}

We can now measure the distance between these simulated results and the
empirical results:

````julia
function euclidean_distance(x1::Vector{T}, x2::Vector{T}) where {T <: AbstractFloat}
  return sum(sqrt.((x1.-x2).^2.0))
end
distances = map(s -> euclidean_distance(s, summary(empirical_data)), simulated_results)
````



````julia
density(distances, leg=false, fill=(0, :orange, 0.4), c=:orange)
xaxis!("Distance", (0,1))
yaxis!("Density", (0, 4))
````


{{< figure src="../figures/abc_13_1.png" title="Distance between the summaries of empirical and actual data."  >}}


The actual ABC step is to *reject* some of the samples from them prior
distribution, to only select the parameters combinations that produce results
*close* to the empirical data. This is done by setting a threshold, and
distances *above* this threshold will be rejected (this is why the choice of a
good prior distribution is crucial). In this case, we picked a threshold value
of $\theta = 0.02$. Why? Well, for the same reason we idolize $p \le 0.05$:
"dunno, it looks fine I guess?". You are, once again, encouraged to change this
value, and the density of distances presented in the previous plot should help
you decide which values are too large or too small; and as always, you can
increase the number of samples if you want to be very strict.

````julia
θ = 0.02
posterior = findall(distances.<θ)
````





{{< callout information >}}
Another approach to rejection sampling would be to keep generating samples until
the sample size of the posterior has reached a certain threshold. Both solutions
are valid, but for the sake of illustration, we went with the one that was
simpler to implement. Feel free to work on other rejection samplers as a
programming exercise.
{{< /callout >}}

````julia
density(Sc, c=:teal, ls=:dash, lab="")
density!(Se, c=:purple, ls=:dash, lab="")
density!(Sm, c=:grey, ls=:dash, lab="")

density!(Sc[posterior], c=:teal, fill=(0, :teal, 0.3), lab="Colonization")
density!(Se[posterior], c=:purple, fill=(0, :purple, 0.3), lab="Extinction")
density!(Sm[posterior], c=:grey, fill=(0, :grey, 0.3), lab="Error")

xaxis!("Parameter value", (0,1))
yaxis!("Density", (0, 14))
````


{{< figure src="../figures/abc_15_1.png" title="Posterior distributions (shaded) and the corresponding priors (dashed) for the three parameters in the model."  >}}


To summarize, we can now extract the values of the different parameters:

|     | meaning                |                                     mean |                      standard deviation |
|:---:|:-----------------------|-----------------------------------------:|----------------------------------------:|
| $c$ | Colonization rate      | 0.555 | 0.067 |
| $e$ | Extinction rate        | 0.154 | 0.041 |
| $m$ | Measurement error rate | 0.051 | 0.039 |

Done! One of the strength of ABC is that we can now *generate* datasets, using
our original model for simulation. And because we know the measurement error
rate, we can also correct these datasets to describe the data *generation*
process (the actual presence/absence of the species) as opposed to the data
*observation* process (measuring the presence/absence and being sometimes wrong
about it).
