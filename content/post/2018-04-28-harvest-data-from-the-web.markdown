---
title: HaRvest data from the web
author: ~
date: '2018-04-28'
slug: harvest-data-from-the-web
categories: ['programming']
tags: []
---

# Intro

There's a cool package called [`rvest`](https://github.com/hadley/rvest) which makes scraping data from the web pretty easy. This package is designed to work with [`magrittr`](https://github.com/tidyverse/magrittr) so users get the benefits that come along with the `%>%` operator.

Let's start with an example of how someone can extract a table of data from wikipedia.

## Sleepy Hollow, NY

We start with a URL, do a bunch of stuff, and ultimately extract a clean data frame. In this example the URL points to a wikipedia page about Sleepy Hollow, New York:


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

Notice that the pipe operator `%>%` allows us to write code in a step by step fashion. Personally, I find pipe syntax intuitive and more natural but you could do this the old fashioned way if you wanted.

This chain has 11 lines of code so let's go over the steps involved.

### `read_html`

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

This function is what actually gives us the html data that is then parsed to xml.

### `html_table` 

After this, `html_table` parses the data into a list of data frames. Note that an additional argument `fill = TRUE` is required in this case. This is because the tables I'm reading have inconsistent dimensions, there is at least one table that has a column with less rows than the others. To resolve this issue, we fill such instances with `NA`.

### Extracting data frames with `[[`

After that, we extract the second table using `[[(2)` to extract the second element in our list of data frames, this happens to be the population data for Sleepy Hollow, NY.

From this point we've got the data frame but some additional prep is needed. Rows need to be deleted and some values need characters removed so that the data types can be corrected. Currently, we have character columns for numeric data. There are a bunch of ways we could go about doing this, many that are probably more elegant than my solution but it's quick and regular expressions are not one of my strengths.

### Data prep

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
  theme_minimal() +
  theme(text = element_text(family = "Source Code Pro"),
        panel.grid.minor = element_blank(),
        panel.grid.major.x = element_blank())
```

<img src="/post/2018-04-28-harvest-data-from-the-web_files/figure-html/unnamed-chunk-6-1.png" width="672" />

# Managing multiple data frames

Most times, there will be numerous tables on a single page. I like to handle these situations by creating a nested data frame. The idea is that you have one data frame that contains data frames. The `tibble` package has a nice function called `enframe` that can do this. 

## List of best selling books

Now let's take a look at a page with multiple tables, read them all, and then store them into a single R object.

### Import


```r
# URL object
books_url <- "https://en.wikipedia.org/wiki/List_of_best-selling_books"
```

### Extract additional variables


```r
# Range object to call map
range <- books_url %>% 
  read_html() %>% 
  html_nodes("h3") %>% 
  .[1:16] %>% 
  .[-11] %>% 
  html_text() %>% 
  str_remove("\\[[^]]*]")
```


```r
# Header object to call map
header <- books_url %>% 
  read_html() %>% 
  html_nodes("h2") %>% 
  .[2:4] %>% 
  html_text() %>% 
  str_remove("\\[[^]]*]") %>% 
  str_to_lower() %>% 
  rep(5) %>% # luckily the data structure supports this
  sort()
```

# Create nested data frame


```r
# Create data frame
books <- books_url %>% 
  read_html() %>% 
  html_table(fill = TRUE) %>% 
  .[2:16] %>% 
  map(rename_all, tolower) %>% 
  map(rename_all, str_remove_all, "[[:punct:]]") %>% 
  map(rename_all, str_replace_all, " ", "_") %>% 
  map(~ mutate(., approximate_sales = str_remove(approximate_sales, "\\[[^]]*]"))) %>% 
  map(~ mutate(., sales = str_extract(approximate_sales, "[[:digit:]]+"))) %>% 
  enframe(name = "id") %>% 
  mutate(range = range, header = header)
```

### Look at data


```r
# Counts for approximate sales, 10 million occurs most often
books %>% 
  filter(id == 5) %>% 
  unnest() %>%
  type_convert() %>% 
  group_by(approximate_sales) %>% 
  count() %>% 
  arrange(desc(approximate_sales))
```

```
## Parsed with column specification:
## cols(
##   range = col_character(),
##   header = col_character(),
##   book = col_character(),
##   authors = col_character(),
##   original_language = col_character(),
##   approximate_sales = col_character(),
##   sales = col_integer()
## )
```

```
## # A tibble: 10 x 2
## # Groups:   approximate_sales [10]
##    approximate_sales                       n
##    <chr>                               <int>
##  1 18 million (in Japan and China)         1
##  2 16 million                              3
##  3 15 million                             10
##  4 14 million                              6
##  5 13 million                              2
##  6 12 million                              6
##  7 11-12 million (during 20th century)     1
##  8 11 million                              2
##  9 10.5 million                            2
## 10 10 million                             22
```





