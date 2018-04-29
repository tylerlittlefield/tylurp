---
title: HaRvest data from the web
author: ~
date: '2018-04-28'
slug: harvest-data-from-the-web
categories: ['programming']
tags: []
---

There's a cool package called [`rvest`](https://github.com/hadley/rvest) which makes scraping data from the web pretty easy. This package is designed to work with [`magrittr`](https://github.com/tidyverse/magrittr) so users get the benefits that come along with the `%>%` operator. 

For example, we could read population data from a wikipedia page:


```r
library(tidyverse)
library(rvest)

# Scrape population data from wikipedia 
"https://en.wikipedia.org/wiki/Sleepy_Hollow,_New_York" %>% 
  read_html() %>% 
  html_table(fill = TRUE) %>% 
  `[[`(2) %>% 
  set_names(c("year", "population","blank", "percentage")) %>%
  select(year, population) %>% 
  as_tibble() %>% 
  filter(year != "Census") %>% 
  mutate(year = str_replace(year, "Est. ", "")) %>% 
  filter(year != "U.S. Decennial Census[12]") %>% 
  type_convert()
```

```
## # A tibble: 15 x 2
##     year population
##    <int>      <dbl>
##  1  1880       2684
##  2  1890       3179
##  3  1900       4241
##  4  1910       5421
##  5  1920       5927
##  6  1930       7417
##  7  1940       8804
##  8  1950       8740
##  9  1960       8818
## 10  1970       8334
## 11  1980       7994
## 12  1990       8152
## 13  2000       9212
## 14  2010       9870
## 15  2016      10198
```

We start with a URL and then call `read_html`:


```r
"https://en.wikipedia.org/wiki/Sleepy_Hollow,_New_York" %>% 
  read_html()
```

```
## {xml_document}
## <html class="client-nojs" lang="en" dir="ltr">
## [1] <head>\n<meta http-equiv="Content-Type" content="text/html; charset= ...
## [2] <body class="mediawiki ltr sitedir-ltr mw-hide-empty-elt ns-0 ns-sub ...
```

After this, `html_table` parses the data into a list of data frames. Note that an additional argument `fill = TRUE` is required in this case. This is because the tables I'm reading have inconsistent dimensions, there is at least one table that has a column with less rows than the others. To resolve this issue, we fill such instances with `NA`.

After that, we extract the second table using `[[(2)` to extract the second element in our list of data frames, this happens to be the population data for Sleepy Hollow, NY.

From this point we've got the data frame but some additional prep is needed. Rows need to be deleted and some values need characters removed so that the data types can be corrected. Currently, we have character columns for numeric data. There are a bunch of ways we could go about doing this, many that are probably more elegant than my solution but it's quick and regular expressions are not one of my strengths.

First thing is to rename the columns. I want them to be lower case and short so I can call on them easily. 


```r
"https://en.wikipedia.org/wiki/Sleepy_Hollow,_New_York" %>% 
  read_html() %>% 
  html_table(fill = TRUE) %>% 
  `[[`(2) %>% 
  set_names(c("year", "population","blank", "percentage")) %>% 
  names()
```

```
## [1] "year"       "population" "blank"      "percentage"
```

After this, I'm going to grab the two columns I'm interested in and then convert the data frame to a `tibble`. I want to see the data types and be reminded of them:


```r
"https://en.wikipedia.org/wiki/Sleepy_Hollow,_New_York" %>% 
  read_html() %>% 
  html_table(fill = TRUE) %>% 
  `[[`(2) %>% 
  set_names(c("year", "population","blank", "percentage")) %>%
  select(year, population) %>% 
  as_tibble()
```

```
## # A tibble: 17 x 2
##    year                      population               
##    <chr>                     <chr>                    
##  1 Census                    Pop.                     
##  2 1880                      2,684                    
##  3 1890                      3,179                    
##  4 1900                      4,241                    
##  5 1910                      5,421                    
##  6 1920                      5,927                    
##  7 1930                      7,417                    
##  8 1940                      8,804                    
##  9 1950                      8,740                    
## 10 1960                      8,818                    
## 11 1970                      8,334                    
## 12 1980                      7,994                    
## 13 1990                      8,152                    
## 14 2000                      9,212                    
## 15 2010                      9,870                    
## 16 Est. 2016                 10,198                   
## 17 U.S. Decennial Census[12] U.S. Decennial Census[12]
```

Finally, `filter` `year` to return all rows except ones that contain `"Census"` thereby removing row one. Replace the `"Est. "` from any row value in `year` with nothing, effectively removing those characters and white space. Then, `filter` once more to remove the last row and use `type_convert` to make `year` integer type and `population` numeric.


```r
"https://en.wikipedia.org/wiki/Sleepy_Hollow,_New_York" %>% 
  read_html() %>% 
  html_table(fill = TRUE) %>% 
  `[[`(2) %>% 
  set_names(c("year", "population","blank", "percentage")) %>%
  select(year, population) %>% 
  as_tibble() %>% 
  filter(year != "Census") %>% 
  mutate(year = str_replace(year, "Est. ", "")) %>% 
  filter(year != "U.S. Decennial Census[12]") %>% 
  type_convert()
```

```
## Parsed with column specification:
## cols(
##   year = col_integer(),
##   population = col_number()
## )
```

```
## # A tibble: 15 x 2
##     year population
##    <int>      <dbl>
##  1  1880       2684
##  2  1890       3179
##  3  1900       4241
##  4  1910       5421
##  5  1920       5927
##  6  1930       7417
##  7  1940       8804
##  8  1950       8740
##  9  1960       8818
## 10  1970       8334
## 11  1980       7994
## 12  1990       8152
## 13  2000       9212
## 14  2010       9870
## 15  2016      10198
```

Make this an object and then plot the data:


```r
sleepy <- "https://en.wikipedia.org/wiki/Sleepy_Hollow,_New_York" %>% 
  read_html() %>% 
  html_table(fill = TRUE) %>% 
  `[[`(2) %>% 
  set_names(c("year", "population","blank", "percentage")) %>%
  select(year, population) %>% 
  as_tibble() %>% 
  filter(year != "Census") %>% 
  mutate(year = str_replace(year, "Est. ", "")) %>% 
  filter(year != "U.S. Decennial Census[12]") %>% 
  type_convert()

sleepy %>% 
  ggplot(aes(year, population)) +
  geom_line() +
  geom_point() +
  labs(title = "Sleepy Hollow") +
  theme_bw() +
  theme(text = element_text(family = "Source Code Pro"))
```

<img src="/post/2018-04-28-harvest-data-from-the-web_files/figure-html/unnamed-chunk-6-1.png" width="672" />



