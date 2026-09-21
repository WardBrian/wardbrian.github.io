---
title: 'The happy path, the special cases, and "the other one"'
date: 2022-11-10
permalink: /blog/2022/11/stan-variadics
tags:
  - stan
  - software engineering
---

This is a post about some recent refactors I did to the [Stan](mc-stan.org)
[compiler](https://github.com/stan-dev/stanc3). The specific problem at hand was
cleaning up the code that the compiler uses to type-check and code-generate the
functions in Stan which are higher-order (they take in other functions as an
argument) and variadic (they have a variable number of arguments), but it is
also an interesting example of a theory of coding I've grown more and more fond
of recently.

I don't have a fun name or acronym for this theory (one may already exist which
covers the same ground), but I like to summarize it as: write special cases
until you have three which look very similar[^1].

## Variadic higher-order functions in Stan

First, a brief overview of the problem. VHOFs (as I will now be calling them)
are useful for a few kinds of program design. For example, the Stan language
exposes a few kinds of ordinary differential equation (ODE) solvers. These
solvers require the user to specify the system as a Stan function, and then
call the solver with that Stan function and some control arguments for initial
states, tolerances, etc. The system function takes, at a minimum, a time
argument and a vector of state. However, it is often useful for additional
information to be available to the calculation in the system function.

Because the Stan language lacks
[closures](https://en.wikipedia.org/wiki/Closure_(computer_programming))
[^2], the only way to get this additional information into the function is with
extra arguments besides `time` and `state`.

### Pack it up
Stan originally solved this problem by having a fixed signature for the system
functions, which looked like

```stan
array[] real ode(real time, array[] real state, array[] real theta,
                 array[] real x_r, array[] int x_i);
```

The arguments after `time` and `state` are additional values, which are grouped
by their types: parameters are in `theta`, real data are in `x_r`, and integer
data are in `x_i`.

The actual integrators then took the system function, the control arguments, and
then the same three additional arguments. These three were passed directly to
the system function.

This of course has some obvious problems. The user is in charge of turning
whatever values they want to pass into these flat arrays, which means some
rather messy code to flatten matrices and concatinate variables, and then the
inverse to get everything back into useable values on the inside of the
function.

On the computational side, this also means that everything used in these
functions is hard-copied, every call.

### Introducing: variadics

The solution to this (ignoring a lot of C++ engineering effort and compiler
enhancements) was to add new integrator functions which accept a variable number
of arguments. To be more specific, a new set of higher-order functions were
introduced with a much more relaxed set of requirements for the system
functions. They only *needed* to take a time, and a state, but they could have
*arbitrary* additional arguments. The signature for the ODE integrator itself
also needed to be arbitrary: It accepted a function, some control arguments, and
then it accept arguments corresponding to the function as the first argument.

This is much more complicated to describe, but much more natural to actually use
in code. If you need an additional matrix in the system function, you just add
it as an extra argument and then pass it at the end of the call to the
integrator.

This trick is so nifty that the number of usages of it grew over time. Besides
adding more ODE integrator, Stan now also has variadic functions for parallel
reductions (`reduce_sum`) and for differential-algebraic equation solvers
(`dae`).

### The complication

There are two main difficulties in the language's implementation of these
functions: type checking and code generation.



## Refactoring




[^1]: This statement can really be treated as a heuristic for when you should ignore Sandi Metz's great advice in [The Wrong Abstraction](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)
[^2]: For now! See Bob Carpenter's [talk at ProbProg 2021](https://statmodeling.stat.columbia.edu/2021/10/22/carpenter-slides-papers-prob-prog-2021/)
