---
title: The flow of execution
weight: 2
contributors:
    - tpoisot
concepts:
    - control flow
    - Booleans
    - iteration
packages:
    - Statistics
---

## Programming really *is* a language

But if you understand three words, you will be able to hold a good conversation
with your computer! These three words are *if*, *for*, and *while*. If you have
some previous experience with writing code, you can skim through this lesson.

One great way to make your code robust is to keep it very simple, and one great
way to keep your code very simple is to recognize that often, we want to do one
of three things: do one thing if something happens (`if`), do one thing to a
series of things (`for`), or do one thing until something happens (`while`).
These three possibilities define what we call the *control flow*, or the *flow
of execution*.

## After this lesson, you will be able to ...

- ... express your problems in Boolean terms (true/false)
- ... create loops and conditionals
- ... navigate in arrays
- ... understand the difference between `for` and `while`

## Tossing coins and planning trips

Let's imagine a situation where we have a coin, and we can toss this coin. One
output of this observation is whether the coin landed on its head, or on its
tail. We can express the outcome of coin toss as a *statement*: "it is true that
the coin landed on its head", or "it is not true that the coin landed on its
head".

This is not how we would think about the outcome as humans. It would be more
natural to say "head" or "tail". But expressing things as *true* or not true
(which we call *false*) is much more easier for computers to understand. A great
deal of programming is finding out ways to reduce the outcomes to *true*/*false*
statements.

In fact, there is a name for this type of data: Boolean. In the Boolean world,
things are either *true*, or *false*, and we decide accordingly. Very often, we
think in Boolean terms without noticing it! For example, when wondering if it is
faster to go to work by bus, or by bike, we are expressing in our own way the
question of "going to work by bus is faster than by bike, true or false?".

And then, we will of course take a decision based on the outcome of this
question. "If it is faster to go by bike, then I will go by bike".

Have you noticed that the word *if* appeared a lot in the past few sentences? It
is because `if` is the first way to control the flow of execution. It is one of
the words that many programming languages already know (we call these
*keywords*), and it lets us decide what to do when confronted with alternative
choices.

Let's say I am sitting in my office, and I need to attend a meeting on the other
side of campus. After looking at the itinerary, I can either bike (4 minutes) or
walk (13 minutes). To decide what to do, I can ask the following question to my
computer:

~~~
walking takes 13 minutes
biking takes 4 minutes
if (walking is faster than biking)
    tell me to walk
~~~

This block above is called *pseudocode*. It is a way to start expressing our
ideas in a language we can understand, but that resembles what the computer
speaks. We will write quite a lot of it.

Now, let's give this a try - before you do, what do you think will happen?

````julia
time_by_foot = 13
time_by_bike = 4
if time_by_foot < time_by_bike
    println("You should walk")
end
````





Uh, weird! Nothing happened.

Let's think about why. We asked the computer to compare the time by foot and the
time by bike; if the time by foot is shorter, then we print a line (`println`)
telling us to walk. But we know that the time by foot is *not* shorter, and so
does the computer. And for this reason, whatever is between `if` and `end` is
*not* executed. Testing that conditions are met are one way to save time -- we
do not want to run operations that are not useful.

In the above example, we gave no alternative to the computer. To decide between
two (or more) things to do, we need to use `if`'s frequent partner: `else`.
Let's try again:

````julia
time_by_foot = 13
time_by_bike = 4
if time_by_foot < time_by_bike
    println("You should walk")
else
    println("You should bike")
end
````


````
You should bike
````





This time, we get the right output: `You should bike`. This brings a very
important point: we need to be *explicit*; when talking with humans, we can
understand (or guess) what the alternative choice is. Computers have no such
abilities: everything that happens is the outcome of things we (or others) have
written in the code.

In practice, we will want to make decisions based on several factors. This is a
thing at which Boolean values excel: we can perform *operations* on them. The
most common ones are *not*, *or*, and *and*.

The *not* operation is, quite literaly, the opposite of a statement. For
example, if we state "it is true that the coin landed on its head", then *not*
this statement is "it is *not* true that the coin landed on its head", which is
the same thing as "it is false than the coin landed on its head".

Most programming languages use `!x` to mean *not x*. If we run the code below,
what you do think will happen?

````julia
println(!true)
````


````
false
````



````julia
println(!false)
````


````
true
````





Adding `!` in front of a statement will return the *other* Boolean value.

Boolean values can also be *combined*. Coming back to deciding on a mode of
transportation: the same trip by subway would take 8 minutes. Biking is still
faster, but what if it is raining? We can add a rule, to say:

~~~
if it rains
    take the subway
else
    if the subway is faster than biking
        take the subway
    else
        take the bike
~~~

This block above is called a *nested* statement. We start with an `if`, and then
*within it*, have another `if`. This is not *too* bad, but increasing the
nestedness of statements is a very effective way of having too much complexity!
And too much complexity is, in turn, a great way to introduce mistakes that are
hard to understand. This is, generally, the opposite of what we want to do.

So we can re-word this expression slightly:

~~~
if (the subway is faster than the bike) or (it rains)
    take the subway
else
    take the bike
~~~

There is a new word here: *or*. The *or* operator will look at both statements
(Is the subway faster? Is it raining?), and return `true` if *either* of them is
true. Let's have a look:


````julia
println("true or false:\t", true || false)
````


````
true or false:	true
````



````julia
println("false or false:\t", false || false)
````


````
false or false:	false
````



````julia
println("false or true:\t", false || true)
````


````
false or true:	true
````



````julia
println("true or true:\t", true || true)
````


````
true or true:	true
````





Most programming languages will use `||` or `or` or `|` to write the *or*
operation. We can now fine tune our code, to decide between the subway and the
bike, as a function of the weather. Run the cell below: what do you expect?


````julia
time_by_subway = 8
time_by_bike = 4
rain = true

if (time_by_subway < time_by_bike) | rain
    println("You should take the subway")
else
    println("You should bike")
end
````


````
You should take the subway
````





Because it rains (`rain = true`), our code is correctly telling us to take the
subway.

Now, what would you change to make it stop raining? And what do you expect to
see as an output?
{: .question}

Correct! If you change the `rain = true` to `rain = false`, and re-run this
example, the code will tell us to bike, because it is faster.
{: .answer}

At this point, it is important to note that there are many, many ways to write
the same code. Maybe you would like to ask the question "Is it *not* raining?"
instead, or decide which mode of transporation takes the longest time. As long
as they give the correct answer, all of these formulations are valid. The
important thing is that they let you write code that is easy to read, and easy
to understand.

What if nested statements are easier to understand for you? Well, this is fine.
The most important thing is to write code that prevents you from making
mistakes. If you are more confident in your nested statements, then use them!

Before we move on, there is a final operation on Booleans we need to discuss:
*and*. Most programming languages will use `&` or `&&` or `and` to describe it.
The *and* operation will look at both statements, and return *true* only if both
are *true*:


````julia
println("true and false:\t", true && false)
````


````
true and false:	false
````



````julia
println("false and false:\t", false && false)
````


````
false and false:	false
````



````julia
println("false and true:\t", false && true)
````


````
false and true:	false
````



````julia
println("true and true:\t", true && true)
````


````
true and true:	true
````





**So far**, we have learned about Boolean values, and the *if* operation. Using
*if* is a way to look at a statement, and do different things when it is true
or false. In a lot of cases, we want to also perform operations on a large
number of elements. To do so, we will use the second word: *for*.

## Introducing iteration

The `for` operation is one of the most common, but also one of the most
confusing, ways to tell a computer what to do. This is because it requires to
understand a lot of concepts at once; we will walk through each of them, get
confused a little bit, then get confused a lot, then get it.

When talking about `for`, we usually talk about *for loops* or *iteration*. This
is because `for` lets you express the fact that you will perform an operation on
a (finite) set of elements. Let's start with a perfectly boring yet somewhat
instructive example. We can draw five random numbers between 0 and 1, using

``` julia
rand(5)
```

We might want to print *foo* when a number is lower or equal to 0.5, and *bar*
in the rest of the situations. Why on Earth would we need to print *foo* and
*bar*? Because these are nonsense words used as placeholders by programmers. In
case you want to get all fancy, these are actually called "metasyntactic
variables". If you want more than two, we suggest *baz*, and then *wibble* and
*wobble*.

````julia
random_numbers = rand(5)

for random_number in random_numbers
  if random_number ≤ 0.5
    println("foo")
  else
    println("bar")
  end
end
````


````
bar
bar
foo
bar
bar
````





There is quite a lot happening here, so we will go line by line.

~~~ julia
random_numbers = rand(5)
~~~

First, we generate 5 random numbers, and put them in a variable called
`random_numbers`. It is always a good idea to give very explicit names to
variables. To begin with, most code editors will be very good at autocompletion:
type a few letters, then hit the *Tab* key, and you will see the possible
values.

Giving plural names to things that have multiple elements is also useful: it
helps to have code that reads like plain english. By contrast, variables whose
name is singular have a single value in them.

Now we can start the loop itself:

```raw
for random_number in random_numbers
    # Content of the loop
end
```

This line gives a simple instruction to your computer. Actually, no. It gives a
bunch of complex instructions to your computer, but it is an easy enough
instruction for us to *write*, and this is all that matters.

It goes something like this:

1. look at what is inside `random_numbers`
1. take the first value, and name it `random_number`
1. do whatever we tell you to do with this variable until you hit `end`
1. move on to the next value of `random_numbers`, and start again
1. when you have exhausted the values in `random_numbers`, continue to whatever is *after* the end of the loop

The *for* loop is one of the most difficult construct to understand, because of
this "change the content of the variable" trickery. We will have a few more
examples in this lesson.

The final lines we need to look at are in the *inside* of the loop -- we call
this inside thing *the body* for no particular reason.

```raw
if random_number ≤ 0.5
    println("foo")
else
    println("bar")
end
```

These lines should be familiar to you now -- your computer will evaluate the
statement "`random_number` is lower than or equal to 0.5", and depending on the
truthiness of it, will print either *foo* or *bar*.

## Navigating arrays

Before we move on to a more interesting use of iteration, it is worth
understanding what exactly is in the `random_numbers` object. Let's display it
again:


````julia
random_numbers
````


````
5-element Array{Float64,1}:
 0.5850840517891143  
 0.5140082053721966  
 0.016099656463364198
 0.657105449169457   
 0.857921938720698
````





This type of object is an *array*; it may help to think of an array as a shelf,
in which every compartment can store one thing. You can have shelves with a
single row, a single column, or both rows and columns. In *Julia*, arrays are by
default columns, and this is important for applications like linear algebra
(they behave as vectors). Arrays have all sorts of properties, the most
important being their *length*:

````julia
length(random_numbers)
````


````
5
````





This tells us that our "shelf" has five compartments, so we can store five
things in it. We can also ask what its *size* is:


````julia
size(random_numbers)
````


````
(5,)
````





The output is `(5,)` - this is the computer way of telling us that this array
has 5 positions in its first *dimension*, and no positions in its second
dimension: this is a column with five rows. We can also access any *position* we
like; this is akin to asking "computer, give me the content of the 1st
compartment":

````julia
random_numbers[1]
````


````
0.5850840517891143
````





Some languages, like *Julia*, *R*, and *MatLab*, start counting from 1, but
*python* and *C* start counting from 0. These are conventions that each language
adopted. Everyone thinks the other camp is wrong, and it's one of these
surprisingly bitter (considering how utterly unimportant they are) divides in
the computer science world.

We can *also ask what the *last* position contains:

````julia
random_numbers[length(random_numbers)]
````


````
0.857921938720698
````





The way to read this instruction is as follows: get me the element at position
`length(random_numbers)`. We know that the length of `random_numbers` is `5`, so
this will return the 5th position. *Julia* has a quite pretty way of getting the
last element of most collections:

````julia
random_numbers[end]
````


````
0.857921938720698
````





An extra bit of [syntactic sugar][sugar] in *Julia* are the two following
functions:

````julia
first(random_numbers)
````


````
0.5850840517891143
````



````julia
last(random_numbers)
````


````
0.857921938720698
````





[sugar]: https://en.wikipedia.org/wiki/Syntactic_sugar

Being able to access elements by their position can be very useful. Our
`random_numbers` array has five elements, and we only want to print the
odd-numbered ones. One way to do this would be to call then individually:


````julia
println(random_numbers[1])
````


````
0.5850840517891143
````



````julia
println(random_numbers[3])
````


````
0.016099656463364198
````



````julia
println(random_numbers[5])
````


````
0.857921938720698
````





Of course, this is only reasonable if we have a very small number of things to
do. But what if we want to iterate over hundreds, or thousands of values? We
need a more efficient strategy.

## Iterating over values

We know that a number is even if the statement `x % 2 == 0`, where `%` is
integer division. We can also say that a number is even if the remainder of its
integer division by two is *not* 0: `x % 2 != 0`.

Let's go:


````julia
for i in 1:length(random_numbers) # We could also write `in eachindex(random_numbers)`
  if i % 2 != 0
    println("Position $i:\t", random_numbers[i])
  end
end
````


````
Position 1:	0.5850840517891143
Position 3:	0.016099656463364198
Position 5:	0.857921938720698
````





We can "read" this snippet (a *snippet* is the affectionate name given to a
litle chunk of code; a *chunk* is a much uglier name for "a piece") as

~~~
there is a variable i
it will take every value between 1 and the length of the ranom_numbers array
for every value
look if it is odd
if it is, print the random number at this position
~~~

All `for` loops will share this structure:

~~~ julia
for element in collection
    # this bit can be as complex as we like -- but not too complex!
    do_things(element)
end
~~~

There is an important notion to mention: the *scope*. The scope is the parts of
your program in which a variable exists. Let's look at this hypothetical code:

~~~ julia
for i in 1:3
    println(i)
end
~~~

It will take the values 1, 2, and 3, and put them in the variable `i`, one at a
time. This is like writing

~~~ julia
i = 1
i = 2
i = 3
~~~

Right? So let's try. What do you think will happen if you run the cell below?


````julia
for i in 1:3
    println(i)
end
````


````
1
2
3
````



````julia

println(i)
````


<pre class="julia-error">
ERROR: UndefVarError: i not defined
</pre>




What happens is that the variable `i` only exists within the `for` loop! This
might seem problematic at first, but it is actually much cleaner: this avoid
polluting your workspace with a lot of variables that are not really relevant.

This is true for all variables created within a loop. In the following code, `a`
is not defined outside of the loop:

~~~ julia
for i in 1:3
    a = i
end
~~~

If you want a variable to be accessible *outside* a loop, you can simply create
it before *and* declare it as a `global` variable (this is not required within
functions, and this is the point where reading the section of the Julia manual
on scope will help you):

````julia
a = 0
for i in 1:3
    global a
    a = i
end
println(a)
````


````
3
````





## Doing something until something happens

Before moving on, there is an additional construct we can use: `while`. This one
is *dangerous* (or at the very least *possibly inconvenient*), because it will
keep on running *until* something happens. Why is not called `until`? Because
most programming languages have been designed with little regard for the way
humans think...

A good example of `while` in action is to keep on generating random numbers
*until* their mean is within a certain range of a desired value. The `rand()`
function will generate numbers uniformly distributed between 0 and 1, so we can
run it for a while (*GET IT?*) to get a sample with an average of about 0.5.

We can for example write it this way:

````julia
using Statistics # We need this to use mean

my_collection = rand(5)

while !(0.499 ≤ mean(my_collection) ≤ 0.501)
  append!(my_collection, rand(5))
end

println("μ: $(round(mean(my_collection); digits=4))")
````


````
μ: 0.4992
````





The instruction just after `while`, *i.e.*

~~~
!(0.499 ≤ mean(my_collection) ≤ 0.501)
~~~

is worth thinking about. It is a very compact way of running multiple tests at
once: the mean needs to be larger than 0.499, yet smaller than 0.501, and we
need to continue *until* this is true (but there is not *until* statement, so we
use the awkward "while not").

{{% callout danger %}}
If we are particularly unlucky, we would never get a sequence of random numbers
that would match this condition! In this case, our computer would stubbornly
keep running until the heat death of the universe (or until it breaks, which in
all likelihood will happen earlier). Writing `while` loops can be a complex
exercice, and it is always good to think about "exit strategies". These will be
discussed in the ["Advanced control flow"]({{< ref
"/primers/advanced_control_flow.md" >}}) primer.
{{% /callout %}}
