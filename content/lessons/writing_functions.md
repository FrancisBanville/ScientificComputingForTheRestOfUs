---
title: Writing functions
concepts:
    - writing functions
    - type stability
    - keyword arguments
contributors:
    - tpoisot
    - notzaki
weight: 3
---

## Good code is small code

In the previous lessons, we have seen how to express problems using `for`, `if`,
and `while`. In this lesson, we will see how it is possible to *wrap* these
instructions in functions. Functions allow you to write code that is modular,
can easily be re-used, and (more importantly for us), can easily be tested,
validated, and fixed.

Throughout this lesson, we will pay attention to decomposing a problem into a
series of small parts. The "[UNIX philosophy][unix]" is a useful guide for this,
and it can be summarized as:

1. Make each function do one thing well
2. Expect the output of most functions to become the input of another function
3. Don't hesitate to throw away and rebuild the clumsy parts

In a sense, writing a program works much like writing a manuscript. Writing a
manuscript is a *big task*. But decomposing the task into paragraphs makes it
more manageable. Having an outline lets you ensure that each paragraph logically
flows into the next one. It also lets you rewrite the paragraph without breaking
the flow of your manuscript. Good practices in scientific writing also apply to
writing code!

[unix]: https://en.wikipedia.org/wiki/Unix_philosophy

## After this lesson, you will be able to ...

- ... create functions to automate and repeat tasks
- ... understand the notions of input and output
- ... understand what type stability is

## What is the value of π anyways?

There is a nifty algorithm to estimate the value of $\pi$. We start by drawing a
square, and the length of its sides is 2. Two what? Two arbitrary units, it
doesn't really matter. The circle inscribed within this circle has a radius of
$r = 1$ (arbitrary unit). And therefore, this inscribed circle has an area of
$\pi \times r^2=\pi$ (arbitrary units squared).

Now let's grab some darts (some *computer simulated* darts). If we throw them
randomly at the square, they have a chance of falling within the circle ($\pi /
4$, which is the area of the circle divided by the area of the square), or
outside of the circle ($1 - \pi / 4$). So if we throw a bunch (it's a technical
term for *some*) of darts ($N$) at the square, we can measure the number $n$
that fall into the circle, and then estimate that $\pi \approx 4\times(n/N)$.

We can decompose this problem into a series of steps. First, we need to generate
a series of darts, with coordinates in $(0,2)$. This makes the center of the
square $(1,1)$. We could have thrown darts in $(-1,1)$ with a center at $(0,0)$;
or in $(5,7)$ with a center at $(6,6)$. This is largely arbitrary, but using
$(0,2)$ will be save us almost four keystrokes!

After the following lesson, it is tempting to write:


````julia
N = 10
darts = []
for i in 1:N
    x,y = rand(2).*2
    push!(darts, (x, y))
end
darts
````


````
10-element Array{Any,1}:
 (1.9225655456880713, 1.658569803007004) 
 (1.6680582698706812, 1.3557236665503827)
 (0.5050215751771776, 1.4784549205481996)
 (1.5405078126322218, 0.613905254034699) 
 (1.0015544363511082, 1.8094555548805933)
 (1.5380589866231302, 1.5077082285774792)
 (1.6688886226961581, 1.6053748725468124)
 (1.9094541174426842, 1.152405518131034) 
 (0.2504165312914206, 0.9547856567875397)
 (1.8288391504264139, 0.4369642853850819)
````





This introduces a few new notations. The `darts = []` instruction creates an
*empty* array. This is not the optimal way of approaching this problem, but
remember the UNIX philosophy: we'll rewrite this awkard part later. Then we use
a `for` loop to throw the darts.

At each step of the loop, we generate a new dart: its coordinates are `rand(2)`
(two random numbers in $(0,1)$), which we multiply by `2` to have them in
$(0,2)$.

Finaly, we *add* this set of coordinates to `darts`. The `push!` function will
take its first *argument*, and add its second argument at the end. Wait, hold
on. What's an argument? It's something you give to a function, to get it to do
its work. We will go into this deeply in a few moments.

{{% callout opinion %}}
Our initial intuition of writing this code "as is", *i.e.* not within a
function, is valid. It is often easier to experiment with different ways of
expressing a problem, then wrap it up nicely in a function.
{{% /callout %}}

This code works. But it is kind of messy. There is a `N` variable, which is
declared even though it is unlikely we will need it (because we can get it from
`length(darts)`, remember?). So we will put this code inside its own function.

In *Julia* (and most other languages), we can declare functions using the
following syntax:

~~~
function functionname(foo, bar)
  baz = foo + bar
  return baz
end
~~~

This will generate a function called `functionname`, with two arguments. Inside
the function, a variable called `baz` will be created, and the function will
return its *value*.

It is wise to think of functions as wood chippers. You can put things in, and
you will get things out, but we strongly advise against going inside: if you
limit the *inputs* to the function to its arguments, and its *effects* to the
`return` statement, your function will have a neat, predictible behavior (that
is, at least, the theory). Functions that only act on their arguments, and only
give you something back through `return`, have no [side effect][se].

*Julia* has a special way of indicating functions *with* side effects: they have
a `!` at the end. Remember `push!` from the previous code block? The `!` at the
end tells you "Hey, you are going to change things *outside* of this function,
be careful". We will write a few functions with side effects later.

[se]: https://en.wikipedia.org/wiki/Side_effect_(computer_science)

### Writing a function to throw darts

But what if we want to change the number of darts we will generate? What if
later, we want to re-use this code for anoter project? What if we don't want to
have variables `N` and `darts` in our global scope? In this case, we will write
a function.

Our function will take an *input*, which is the number of darts we want to
throw, and return an *output*, which is an array of coordinates of where the
darts hit. The block below presents *one possible* implementation of such a
function, with comments about whats is hapenning in every line:


````julia
function throw_darts(n::Int64)
    #=
    Let's create a space to store the results in

    The line below will "pre-allocate" a variable called darts.
    Its type is an array, with a single dimension, which contains
    tuples of two floating point numbers.

    Pre-allocation is useful when the size of objects you maipulate
    increases. It tells the computer to reserve enough space in memory.
    Also specifying the type of the objects will let the computer
    reserve exactly enough space in memory.
    =#
    darts = Array{Tuple{Float64,Float64},1}(undef,n)

    #=
    Now we can fill the variable with random darts. Julia has
    a nice function called `eachindex`, which is able to iterate
    over all positions in an array (or more broadly, over a
    collection). Using eachindex(darts) is equivalent to
    writing 1:length(darts).
    =#
    for i in eachindex(darts)
        #=
        This line uses a little bit of julian notation:
        specifically, adding ... after a variable will
        "unpack" its elements, which lets us store them
        in a tuple. The same thing can be done with

        x, y = rand(2).*2
        darts[i] = (x, y)
        =#
        darts[i] = Tuple(rand(2).*2.0)
    end

    #=
    Finally, we explicitly return the value of darts. This is
    not mandatory (by default julia returns the output of the last
    operation that was done within the function), but being explicit
    is always better than being implicit.
    =#
    return darts
end
````


````
throw_darts (generic function with 1 method)
````





Now, we can *call* this function, using `throw_darts(10)`. You can experiment
changing `10` by other values. For example, how do you throw 124 darts?

### How many darts are within the circle?

Now that we are able to throw darts, we can see how many of them are within the
circle. Remember, the circle has a diameter of two, and is centered on $(1, 1)$.
It would be tempting to take *all* the darts we generated, and see how many of
them are within the circle. But it is almost always better to write functions
that do the simplest thing possible: in this case, is a single dart within the
circle?

This is done by measuring the distance between the darts and the center of the
circle -- if it is lower than the radius, then the dart is within the circle. We
can calculate the distance between points $(x_1, y_1)$ and $(x_2, y_2)$ as
$\sqrt{(x_1-x_2)^2+(y_1-y_2)^2}$. This is quite easy to write as a function!

We can decide that this function will accept as input a first point, another
point, and a radius. So the first line will look like:

    function is_within_circle(point, center, radius)

Then, we can take the distance between the points as

    distance = (point[1]-center[1])^2 + (point[2]-center[2])^2

, and so the point is within the circle if

    distance < radius

Let's wrap this up!


````julia
function is_within_circle(point, center, radius)
  distance = (point[1]-center[1])^2 + (point[2]-center[2])^2
  return distance <= radius
end
````


````
is_within_circle (generic function with 1 method)
````





There is a more *idiomatic* way of writing this function -- *idiomatic* means
that we are using some language-specific ways of expressing operations. In this
case, it could be

~~~
function idiomatic_is_within_circle(point, center, radius)
  return sum((point.-center).^2) < radius
end
~~~

{{% callout opinion %}}
Writing things in an idiomatic way is *sometimes* better. But it can be more
confusing for people with less familiarity with the language, or more difficult
to maintain (unless profusely commented) in the long term.
{{% /callout %}}

Now, let's apply a few sanity checks to this function (we will see much more of
this in the next lesson). We know that the center of the circle is within the
circle, that a point above the center at a distance equal to the radius is
within the circle, and that a point above the center as a distance greater than
the radius is not:


````julia
center = (1.0, 1.0)
radius = 1.0
println("Case 1: ", is_within_circle(center, center, radius))
````


````
Case 1: true
````



````julia
println("Case 2: ", is_within_circle((1.0, 2.0), center, radius))
````


````
Case 2: true
````



````julia
println("Case 3: ", is_within_circle((1.0, 2.1), center, radius))
````


````
Case 3: false
````





Although these are very simple tests, it seems that our function checks out. So
we can move forward. But before we do so, let us take a step back.

So far, we have

1. A way of throwing random darts
2. A way of knowing which of these darts fall within a circle

The next step is therefore to take each dart, in turn, and see if they are
within the circle! We can do this with a loop. There are a few ways of doing it.

The first is to pre-allocate an array of Boolean values (`true`/`false`), with
as many spots as we have thrown darts, and store the result of the question *Is
this dart within the circle?*.

Another is to have a counter, starting at `0`, and we add `1` every time a dart
is within the circle.

There are probably about another million different ways to do this. Here we will
write a function using the counter:


````julia
function how_many_darts(darts, center, radius)
    #=
    Our function will take three inputs:
    1. the darts we generated
    2. the center of the circle
    3. the radius of the circle
    =#

    #=
    We create a variable called `n`, and give it a value of `0`
    =#
    n = 0

    #=
    This loop uses the `for element in collection` syntax: the variable
    `dart` will take every vale in `darts` in turn.
    =#
    for dart in darts

        #=
        Only when the dart is within the circle do we
        add 1 to the counter:
        =#
        if is_within_circle(dart, center, radius)
            n = n + 1
        end
    end

    #=
    Finally, we explicitly return the value of n
    =#
    return n
end
````


````
how_many_darts (generic function with 1 method)
````





And we're done! Remember, our approximation of $\pi$ is $4\times(n/N)$, where
$N$ is the number of darts, and $n$ is the number of hits. So we can finally
write a function, specifying $N$, and start to estimate the value of $\pi$:


````julia
function estimate_pi(N)
  center = (1.0, 1.0)
  radius = 1.0
  darts = throw_darts(N)
  n = how_many_darts(darts, center, radius)
  return 4*n/N
end
````


````
estimate_pi (generic function with 1 method)
````





Our function is *building* on the previous, small, functions we wrote. This
results in code that is easy to read (we don't have to scroll through multiple
screens to see what the function does). Because we have named our functions
explicitely, the code is also easy to understand. In the next lesson, we will
see that modular code is easy to *test* (and to debug).

So, how good is our estimate?

````julia
estimate = estimate_pi(100000)
println("Estimate: $(estimate)\t\tπ: $(π+0)")
````


````
Estimate: 3.14016		π: 3.141592653589793
````





## Keyword arguments and default values

In all functions thus far, we *had* to give all the values of all arguments.
Sometimes, it make sense to have some arguments take a default value, which we
can change when we need to, but will remain constant otherwise.

{{% callout warning %}}
Giving default values can save some time, but it means you don't need to *think*
quite as much about the parameters you are using. And thinking is a pretty big
part of research. For this reason, we think that default values should not be
used for parameters that are central to the analysis, but should instead be
reserved for things like filenames, number of iterations, etc.
{{% /callout %}}

Let's say we want to write a quite poorly thought-out function to get the
logarithm of any number in any base. *Julia* has a `log` function, which can be
called with `log(b,n)` to get $\text{log}_b(n)$. We would like to write a
function to get the $\text{log}_2$ by default, but still be able to change the
base if we need.

````julia
function my_log_function(n; b=2.0)
  return log(b,n)
end
````


````
my_log_function (generic function with 1 method)
````





We have declared the arguments of this function differently: `n` is a
*non-keyword* argument, because it makes no sense to fix a default value. On the
other hand, `b` has a default value of `2.0`. If we call this function using,
for example,

````julia
my_log_function(2.0)
````


````
1.0
````





then the computer will see that we have not asked for a specific value of `b`,
and use the default. But if we use the following syntax,

````julia
my_log_function(2.0; b=ℯ)
````


````
0.6931471805599453
````





then the value of `b` will be fixed (to $e$), and the function will use this.

{{% callout opinion %}}
Some languages allow/require to use `;` to separate keywords and non-keywords
arguments. It is a good idea, and we recommend using it even when *calling* the
functions: `my_log_function(2.0; n=2.0)` -- this makes it clear to see where the
keywords arguments are.
{{% /callout %}}

## A note on type stability

An important but often overlooked notion is "type stability". In a nutshell, it
means that within a function, the variables do not change their type. John Myles
White wrote a [great blog post about it][jmw], which I encourage you to read
after going through this lesson. Chris Rackauckas also wrote about [common
performance issues][crcpi], which features type stability.

[jmw]: http://www.johnmyleswhite.com/notebook/2013/12/06/writing-type-stable-code-in-julia/
[crcpi]: http://www.stochasticlifestyle.com/7-julia-gotchas-handle/

The issue with computers (well, one of) is that they give types to everything.
While for us, 2, 2.0, and 2.000000...00 are the same thing, this is not the case
for our computers. And we need to pay attention to this fact to write code that
is reasonably fast. Let's start with a few illustrations:


````julia
typeof(0)
````


````
Int64
````



````julia
typeof(0.0)
````


````
Float64
````





The `typeof` function tells us how the computer is thinking about its argument:
`0` is an integer number, and `0.0` is a floating point number.

Now, if I were to ask you what the sum of 0 and 1.0 is, you would not take
longer than if I has asked you what the sum of 0.0 and 1.0 is. Is it true for
the computer?



````julia
time_int_and_float = @elapsed 0 + 1.0
time_float_and_float = @elapsed 0.0 + 1.0
println("It takes $(round(time_int_and_float/time_float_and_float; digits=2)) times longer to work with different types!")
````


````
It takes 1.3 times longer to work with different types!
````





The exact number may vary, but on my machine, it takes about 1.5 times longer to
do the operation with different types. Why is that?

It is because the computer needs to "rethink" about one of these numbers (*i.e.*
to change `0` in `0.0`, or `1.0` in `1`), before it can do the actual operation.
There is no obvious way to identify functions in which arguments will change
type over time. This being said, writing short functions, and thinking about
what goes in and out, will help catch some mistakes.

And speaking of catching mistakes.... the next lesson will be focused on making
sure our code does what we want it to do. We will discuss defensive programming
and a little bit of testing.
