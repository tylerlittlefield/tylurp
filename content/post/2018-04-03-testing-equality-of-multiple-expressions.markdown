---
title: Testing equality of multiple expressions
author: ~
date: '2018-04-03'
slug: testing-equality-of-multiple-expressions
categories: ['rstats']
tags: ['rstats', 'testing']
description: ''
---



Credit to BrodieG: https://stackoverflow.com/questions/27325005

This post is essentially an extension of a previous post I [wrote](https://tylurp.rbind.io/2018/02/13/measuring-performance/). The only addition is a cool idea for testing near equality of outputs. I'm calling this an "equality matrix", a matrix of methods that displays whether or not they are equal to each other. One use case is benchmarking. As we benchmark multiple solutions to a single problem, testing whether or not the outputs are equal becomes more time consuming as the `all.equal` function only takes two solutions at a time.

Consider the following problem: _extract all numbers in a vector that have a non zero value after the decimal._

We could do this a few ways:


```r
# vector
x <- c(0.0, 0.5, 1.000, 1.5, 1.6, 1.7, 1.75, 2.0, 2.4, 2.5, 3.0, 74.0)
# create objects for testing equality of output
integer_method <- x[as.integer(x) != x]
trunc_method <- x[trunc(x) != x]
round_method <- x[round(x) != x]
mod_method <- x[x %% 1 != 0]
floor_method <- x[floor(x) != x]
```

Now instead of testing every combination to make sure the outputs are equal we can create a matrix that tests all possible combinations at once:




```r
# create an equality matrix
methods_vec <- c("integer_method", "trunc_method", "round_method", "mod_method", "floor_method")
objs <- mget(methods_vec)
outer(objs, objs, Vectorize(all.equal))
#>                integer_method trunc_method round_method mod_method floor_method
#> integer_method           TRUE         TRUE         TRUE       TRUE         TRUE
#> trunc_method             TRUE         TRUE         TRUE       TRUE         TRUE
#> round_method             TRUE         TRUE         TRUE       TRUE         TRUE
#> mod_method               TRUE         TRUE         TRUE       TRUE         TRUE
#> floor_method             TRUE         TRUE         TRUE       TRUE         TRUE
```

That's it! With just a few lines of code we can test the equality of multiple solutions and print the result nicely.
