---
title: 'Making Merges Go Through Using tidylog and anti_join'
date: 2021-06-31
permalink: /posts/2021/06/making-merges-go-through/
output: 
  md_document:
    variant: gfm
tags:
  - R
  - merging
  - joining
  - tidylog
  - anti_join
---

Packages in the `tidyverse` suite, including `dplyr`, represent amazing
contributions data science. One thing that constantly vexed me, though,
was the inability of `dplyr`’s [merge
functions](https://dplyr.tidyverse.org/reference/join.html)–`left_join`,
`full_join`, etc.– to actually let you know if the merges went through
without having to do extra work. It just seemed so cumbersome to have to
individually inspect each data frame after each merge. Well, thankfully,
that is no longer necessary due to `tidylog`.

To explain how `tidylog` helps with [ensuring merges go
through](https://cran.r-project.org/web/packages/tidylog/readme/README.html)
as desired, let’s start by loading two data frames, `df1` and `df2`.
First, let’s load and inspect `df1`:

``` r
# create df1
df1 = data.frame(country = c("Brazil", "USA"),
                 year = c(2020,2020),
                 population_millions = c(213,331))

# inspect df1
head(df1)
```

    ##   country year population_millions
    ## 1  Brazil 2020                 213
    ## 2     USA 2020                 331

Then, let’s load and inspect `df2`:

``` r
# create df2
df2 = data.frame(country = c("Brasil", "USA"),
                 year = c(2020,2020),
                 poverty_rate = c(21,9))

# inspect df2
head(df2)
```

    ##   country year poverty_rate
    ## 1  Brasil 2020           21
    ## 2     USA 2020            9

As should be apparent, they are both data frames with demographic
information on the poverty rates and population of the US and Brazil in
the year 2020. Using the `dplyr` package, we could merge `df1` and `df2`
together using `left_join`:

``` r
# suppress warnings (unnecessary, but makes the script cleaner)
options(warn=-1)
options(tidyverse.quiet = TRUE)

# load library
library(tidyverse)

# merge
initial_merged_df = left_join(df1, df2, by=c("country", "year"))

# inspect the data frame
head(initial_merged_df)
```

    ##   country year population_millions poverty_rate
    ## 1  Brazil 2020                 213           NA
    ## 2     USA 2020                 331            9

As is clear from above, the poverty rate data is `NA` for Brazil. Why
did this happen? The `tidylog` package helps us answer that without even
having to inspect the data frame, which is really useful when working
with larger data frames. To better understand the benefits of `tidylog`,
let’s re-run the merge, but this time let’s load the `tidylog` package
first:

``` r
# load tidylog
library(tidylog, warn.conflicts = FALSE)

# re-run the merge
initial_merged_df = left_join(df1, df2, by=c("country", "year"))
```

    ## left_join: added one column (poverty_rate)
    ##            > rows only in x   1
    ##            > rows only in y  (1)
    ##            > matched rows    (1)
    ##            >                 ===
    ##            > rows total       1

Loading the `tidylog` package lets us know that our merge did not fully
go through. Which observations? To figure that out, let’s use
`anti_join`, which tells us which observations from `df2` did not merge
over to `df1`. To do that, just flip the order of `df1` and `df2` as
follows:

``` r
# checking for problematic observations with anti_join
checking = anti_join(df2, df1, by=c("country", "year"))
```

    ## anti_join: added no columns
    ##            > rows only in x   1
    ##            > rows only in y  (1)
    ##            > matched rows    (1)
    ##            >                 ===
    ##            > rows total       1

Now, let’s take a peak at the data frame:

``` r
head(checking)
```

    ##   country year poverty_rate
    ## 1  Brasil 2020           21

As we can clearly see, the Brazil observation has been spelled with an
“s” instead of an “z”. Because we are working in English, let’s correct
the spelling in English:

``` r
# correct the spelling
df2$country[df2$country=="Brasil"] = "Brazil"
```

Now, we can re-run the merge for a final time, and everything will go
through:

``` r
# final time
final_merged_df = left_join(df1, df2, by=c("country", "year"))
```

    ## left_join: added one column (poverty_rate)
    ##            > rows only in x   0
    ##            > rows only in y  (0)
    ##            > matched rows     2
    ##            >                 ===
    ##            > rows total       2

Now, let’s inspect the data frame:

``` r
head(final_merged_df)
```

    ##   country year population_millions poverty_rate
    ## 1  Brazil 2020                 213           21
    ## 2     USA 2020                 331            9

Clearly, `tidylog` was not 100% necessary for merging the toy data
frames above, `df1` and `df2`. With larger data frames, though,
combining `tidylog` and `anti_join` will be crucial to ensuring that all
of your data actually merge.
