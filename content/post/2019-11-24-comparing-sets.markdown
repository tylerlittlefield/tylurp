---
title: Comparing sets
author: ~
date: '2019-11-24'
slug: comparing-sets
categories: ['rstats']
tags: ['rstats', 'vectors', 'sets']
description: ''
---



## Introduction

The task of comparing sets (unique vectors) is a common one, we often want to confirm whether two things are the same or not. For example, we might rewrite a function to make it run faster, in this case we want to make sure that the new and old function output the same result. On the other hand, we might rewrite a function to fix a bug, in this context we want to make sure the output of the new function is different, potentially resolving the bug and producing the correct output. In this post, I'll go over a little function I made for comparing sets but before I do, let's get some terminology out of the way:

* **Sets:** A set is basically a unique vector.
* **Vectors:** A vector is just a series of values that share the same type (character, logical, etc.).

We must keep in mind whether or not our method of comparison considers unique values. If we want to confirm two vectors are the same, duplicates and all, you wouldn't want to use a function like `setdiff` for example. This is why the terminology above is important.

## Compare

Recently, I needed to come up with a function to compare one set to another set and output the the difference. The goal is to say something like:

> These values are in the current output but not the older/pre-existing output. 

Since I am only concerned in the unique values, `setdiff` sounds like a good approach to this problem. However, there is one caveat... I constantly forget how `setdiff` works. Is it comparing what **X** doesn't have in **Y**? The other way around? My inability to remember is what prompted the creation of a function I call `compare` and it seems others share my experience.

<!--html_preserve-->{{% tweet "1197634755430367235" %}}<!--/html_preserve-->

The function is dead simple, store some common set functions in a list with each list elements name describing the output.


```r
compare <- function(x, y) {
  list(
    "These values are in X but not Y" = setdiff(x, y),
    "These values are in Y but not X" = setdiff(y, x),
    "These values are shared between X and Y" = intersect(x, y),
    "Combined, X and Y return these values" = union(x, y)
  )
}
```

Now, if we want to compare two sets: 

* 1, 2, 3, 4
* 3, 4, 5, 6

We just plug those sets into `compare`:


```r
x <- 1:4
y <- 3:6
compare(x, y)
#> $`These values are in X but not Y`
#> [1] 1 2
#> 
#> $`These values are in Y but not X`
#> [1] 5 6
#> 
#> $`These values are shared between X and Y`
#> [1] 3 4
#> 
#> $`Combined, X and Y return these values`
#> [1] 1 2 3 4 5 6
```

That's it! No more googling what the output of `setdiff` is giving me, just read the name of the list element to be reminded.
