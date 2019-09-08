---
title: "Ex: Recommendation Engine Using Association Rules (Market Basket) in R"
date: 2018-01-13T18:30:12+03:00 
draft: false
description: ""
tags:
categories: ["datascience", "recommendation", "r"]
type: post
url:
author: "Mert Nuhoglu"
output: html_document
blog: datascience.mertnuhoglu.com
resource_files:

-

path: ~/projects/study/ds/recommendation_engine_using_association_rules_in_r.md

---

<!--more-->

```r
library(lubridate, warn.conflicts = F)
library(hms, warn.conflicts = F)
library(tidyr, warn.conflicts = F)
library(urltools, warn.conflicts = F)
library(stringr, warn.conflicts = F)
library(readr, warn.conflicts = F)
library(broom, warn.conflicts = F)
library(RcppRoll, warn.conflicts = F)
library(tibble, warn.conflicts = F)
library(dplyr, warn.conflicts = F)
```

This document uses some of the code explained in [Introduction to Association Rules in R by Yosuke Yasuda](https://blog.exploratory.io/introduction-to-association-rules-market-basket-analysis-in-r-7a0dd900a3e0)

```r
.libPaths("/Users/mertnuhoglu/.exploratory/R/3.4")

library(exploratory)

# Data Analysis Steps
readr::read_csv("/Users/mertnuhoglu/projects/study_data/ds/article_introduction_to_association_rules_in_r_by_yosuke_yasuda_groceries.csv", , quote = "\"", skip = 0 , col_names = FALSE , na = c("","NA")) %>%
  dplyr::mutate(transaction_id = row_number()) %>%
  tidyr::gather(key, product, -transaction_id, na.rm = TRUE, convert = TRUE) %>%
  dplyr::arrange(transaction_id) %>%
  exploratory::do_apriori(product, transaction_id, min_support = 0.0001) %>%
  dplyr::filter(support > 0.0004) %>%
  dplyr::group_by(rhs) %>%
  dplyr::top_n(3, lift)
```


