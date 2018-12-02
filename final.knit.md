---
title: "FINAL"
author: "Baoyi Shi"
date: "2018年12月2日"
output: html_document
---


```r
library(tidyverse)
```

```
## -- Attaching packages ------------------------------------------------------------ tidyverse 1.2.1 --
```

```
## √ ggplot2 3.1.0     √ purrr   0.2.5
## √ tibble  1.4.2     √ dplyr   0.7.8
## √ tidyr   0.8.2     √ stringr 1.3.1
## √ readr   1.1.1     √ forcats 0.3.0
```

```
## -- Conflicts --------------------------------------------------------------- tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(dplyr)
library(readr)
library(ggplot2)
```


```r
#import tree data, retain only trees alive and good to health
tree_df = read_csv("./data/2015StreetTreesCensus_TREES.csv") %>%
  janitor::clean_names() %>%
  filter(status == "Alive", health == "Good") 
```

```
## Parsed with column specification:
## cols(
##   status = col_character(),
##   health = col_character(),
##   zipcode = col_integer(),
##   zip_city = col_character(),
##   boroname = col_character(),
##   Latitude = col_double(),
##   longitude = col_double()
## )
```


```r
tree_df %>%
  group_by(boroname) %>%
  summarise(total = n())
```

```
## # A tibble: 5 x 2
##   boroname       total
##   <chr>          <int>
## 1 Bronx          66603
## 2 Brooklyn      138212
## 3 Manhattan      47358
## 4 Queens        194008
## 5 Staten Island  82669
```
