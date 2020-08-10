---
title: My first CRAN package
author: ~
date: '2019-02-12'
slug: my-first-cran-package
categories: ['rstats']
tags: ['rstats', 'package', 'development', 'CRAN']
description: ''
---



<img src="/post/2019-02-12-my-first-cran-package_files/figure-html/unnamed-chunk-2-1.png" width="100%" />

> Thanks, your package is on its way to CRAN.

## Introduction

I am really excited to announce (12 days late) that [`lisa`](https://cran.r-project.org/web/packages/lisa/index.html) is on CRAN, my first R package available on the [Comprehensive R Archive Network](https://cran.r-project.org/)! This was a pretty big deal for me because I have wanted to submit something to CRAN for a couple of years now.

Anyway, `lisa` is a color palette pacakge, it provides R users with 128 color palettes based on artwork from the worlds greatest artists. These palettes were made by designers, artists, museum curators, and masters of color theory. You can view them all at [colorlisa](http://colorlisa.com/), a beautiful website created by Ryan McGuire.

## Package Details

The `lisa` package contains a list of palettes, a dataset containing palette information and a function for calling and modifying palettes:


```r
str(lisa[1])
#> List of 1
#>  $ JosefAlbers: 'lisa_palette' chr [1:5] "#D77186" "#61A2DA" "#6CB7DA" "#b5b5b3" ...
#>   ..- attr(*, "name")= chr "JosefAlbers"
#>   ..- attr(*, "work")= chr "Adobe (Variant): Luminous Day"

head(artwork)
#>              artist          palette                              work
#> 1      Josef Albers      JosefAlbers     Adobe (Variant): Luminous Day
#> 2      Josef Albers    JosefAlbers_1 Homage to the Square (La Tehuana)
#> 3 Gretchen Albrecht GretchenAlbrecht                      Golden Cloud
#> 4       Billy Apple       BillyApple                           Rainbow
#> 5       Per Arnoldi       PerArnoldi                              Spar
#> 6      Milton Avery      MiltonAvery        Bicycle Rider By The Loire

lisa_palette("JosefAlbers", 2, "discrete")
#> [1] "#D77186" "#61A2DA"
```

There are also two S3 methods, one for printing and one for plotting objects of class `lisa_palette`:


```r
class(lisa$`Jean-MichelBasquiat`)
#> [1] "lisa_palette" "character"

plot(lisa$`Jean-MichelBasquiat`)
```

<img src="/post/2019-02-12-my-first-cran-package_files/figure-html/unnamed-chunk-4-1.png" width="100%" />

That's pretty much it! Currently, `ggplot2` usage requires something like `scale_color_manual`. I've thought about including custom `ggplot2` color functions, similiar to what [`paletteer`](https://github.com/EmilHvitfeldt/paletteer) does but have put this on hold, maybe the next release.


```r
library(ggplot2)

ggplot(mtcars, aes(mpg, disp)) + 
  geom_point(aes(col = factor(gear)), size = 3) + 
  scale_color_manual(values = lisa$`Jean-MichelBasquiat`) + 
  theme_bw()
```

<img src="/post/2019-02-12-my-first-cran-package_files/figure-html/unnamed-chunk-5-1.png" width="100%" />

## CRAN Submission Process

I submitted the package, got rejected with correction comments, resubmitted, and the package was on CRAN the next day. This all took a total of 5 days. My first submission was rejected because I had redundant wording and didn't include a reference to colorlisa. Redundant wording was things like _"This is an **R** package..."_. This type of wording in the description is considered redundant because the user already knows its an R package. 

## Takeaways

If you're like me and want to submit a package to CRAN for the first time I would recommend a couple things:

1. `devtools` and `usethis` are necessary! They actually aren't but they make the development process much easier and enjoyable. I submitted my pacakge using `devtools::release` which introduced me to a whole bunch of checks that I hadn't thought of. For example, spell check, rhub, and more.
2. Test your code with `testthat`, check coverage with `covr`, continously integrate your code with services like travis-ci and appveyor.
3. When possible, make your package [lightweight](http://www.tinyverse.org/), the less dependencies the better.
4. Tweet about your package! No one will use it if they don't know it exists.
5. Read relevant sections of https://r-pkgs.org/
