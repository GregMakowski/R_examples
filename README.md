# R_examples
# =====================================================================================================================
---
title:   "data.table.examples"
author:  "Greg Makowski"
date:    "October 3, 2017"
output:  html_document
source:  https://s3.amazonaws.com/assets.datacamp.com/blog_assets/datatable_Cheat_Sheet_R.pdf
         https://rstudio-pubs-static.s3.amazonaws.com/52230_5ae0d25125b544caab32f75f0360e775.html (join)

minor changes:  change to lower case data table and column names for easier typing
          add comments
          reset the data set after changes, before next section
          extend with join examples
          
---
# =====================================================================================================================
# ==== MAIN ====

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

```{r  include = FALSE}
# ---------------------------------------------------------------------------------------------------------------------
# ---- setup
require(rmarkdown)
require(data.table)
require(Hmsic)
require(knitr)
```


```{r}
set.seed(45L)                      # Create a data.table
dt <- data.table( v1 = c(1L,2L),  # and call it DT
                  v2 = LETTERS[1:3],
                  v3 = round(rnorm(4),4),
                  v4 = 1:12)

dt
```

```{r results='asis'}
# see doc in http://rmarkdown.rstudio.com/authoring_rcodechunks.html 
knitr::kable(dt)
```

# =========================================================
# ==== Subsetting Rows Using i ============================
```{r}
dt[3:5, ]
```

# don't need to use the comma, defaults to the ith position
```{r}
dt[3:5]
```

```{r}
dt[v2=="A"]    # select rows that have value A in column V2
```



```{r}
dt[v2 %in% c("A", "C") ]    # select rows that have value A or C in column V2

```

```{r}
dt[ between(v3, -.35, 35)]
```

```{r}
dt[ !between(v3, -.35, 35)]
```


# =========================================================
# ==== Manipulating on COlumns in j =======================
```{r}
dt[,v2]       # select a column, return as a vector
```

```{r}
dt[, .(v2,v3) ]   # select 2 columns, use .() to return as a data.table
```

```{r}
dt[, sum(v1)]    # return the sum of a column, as a vector
```

```{r}
dt[, .(sum(v1), sd(v3))]  # sum of v1, std dev of v3, returned as a data.table
```

```{r}
dt[, .(Aggregate = sum(v1),
       Sd.v3 =     sd(v3)  ) ]  # create two named cols in new data table
```

```{r}
dt[, .(v1, sd.v3 = sd(v3))]   # select col v1,  select sd(v3).  Because sd.v3 has one value, it is recycled to match length of v1
```

```{r}
dt[, .(print(v2),      # print v2    (multiple output)
       plot(v3) )]     # plot v3
```

# =========================================================
# ==== Doing j by Group ===================================

```{r}
dt[, .(v4.sum = sum(v4)), by=v1]   # calc sum of v4 for every group in v1
```



```{r}
dt[, .(v4.sum = sum(v4)), by = .(v1,v2)]  # sum by categories in 2 cols
```

```{r}
# ---- DBC, with avg of target and count of sample size
dt[, .(dbc_v1_v2 = mean(v3),
       dbcN_v1_v2 = .N),     by = .(v1,v2)]  # aggregate by categories in 2 cols
```



```{r}
dt[, .(v4.sum = sum(v4)), by = sign(v1-1)]  # sum by ...
```

```{r}
dt[, .(v4.sum = sum(v4)), by = .(v1.01 = sign(v1-1)) ]  # sum by ... (rename by-group var calc)
```


```{r}
dt[1:5, .(v4.sum = sum(v4)), by = v1 ]  # sum by ... (after subset rows, like SQL HAVING clause)
```

```{r}
dt[, .N, by = v1 ]  # sum by ... (count rows in groups)
```


# =========================================================
# ==== Adding / updating col by ref in j, using := ========

```{r}
dt[, v1 := round(exp(v1), 2) ]   # update an existing col, in the data table
dt
```

```{r}
# ---- first, reset the dt
dt <- data.table( v1 = c(1L,2L),  # and call it DT
                  v2 = LETTERS[1:3],
                  v3 = round(rnorm(4),4),
                  v4 = 1:12)
dt  # before

# ---- modify multiple columns, in-place or overwriting their prior values
#      assign to a list of two var names, v1 and v2
dt[, c("v1", "v2") := list(round(exp(v1),2),
                           LETTERS[4:6]) ]
dt   # after
```

```{r}
dt[, v1 := NULL]   # delete a col
names(dt)

dt[, c("v3","v2") := NULL]  # delete multiple cols
names(dt)
```

```{r}
# ---- first, reset the dt
dt <- data.table( v1 = c(1L,2L),  # and call it DT
                  v2 = LETTERS[1:3],
                  v3 = round(rnorm(4),4),
                  v4 = 1:12)
names(dt)  # before

cols.chosen <- c("v1", "v2")
dt[, cols.chosen := NULL]     # delete a list of cols, from a variable list
  # as expected, the above does not work.  It adds the col "cols.chosen", then delets it
names(dt)  # after
```


```{r}
# ---- first, reset the dt
dt <- data.table( v1 = c(1L,2L),  # and call it DT
                  v2 = LETTERS[1:3],
                  v3 = round(rnorm(4),4),
                  v4 = 1:12)
names(dt)  # before

cols.chosen <- c("v1", "v2")
dt[, (cols.chosen) := NULL]     # delete a list of cols, from a variable list
  # fix: add () around cols.chosen
names(dt)  # after

```

# =========================================================
# ==== Indexing and Keys ==================================
```{r}
# ---- first, reset the dt
dt <- data.table( v1 = c(1L,2L),  # and call it DT
                  v2 = LETTERS[1:3],
                  v3 = round(rnorm(4),4),
                  v4 = 1:12)
names(dt)  # before

setkey(dt, v2)
dt["A"]         # with index on, row select defaults to key field, like v2=="A"
```


```{r}
dt[c("A", "C")]   # select where (v2 == "A") or (v2 =="C")
```

```{r}
dt["A",         mult = "first"]   # use the multiple param.  Return just the first of each group

dt[c("A", "C"), mult = "first"]   # use the multiple param.  Return just the first of each group
 
dt[c("A", "C"), mult = "last"]    # use the multiple param.  Return just the LAST of each group 
# mult = "all" is the default.  Only valid values are first, last and all

```

```{r}
dt[ c("A", "D") ]               # return all, even if there is no match to a list member (other cols have NA)

dt[ c("A", "D"), nomatch=0 ]    # do NOT return the records that do notmatch 
```

```{r}
dt[ c("A", "D"), sum(v4) ]              # returns NA because no rows with D exist.  Sum(3, 4, NA) is NA

dt[ c("A", "D"), sum(v4), nomatch=0 ]   # be more robust, just sum over valid rows

dt[ c("A", "C"), sum(v4) ]              # "A" and "C" are grouped into the same sum
```


```{r}
dt[ c("A","C"),  sum(v4),  by=.EACHI]    # operate by each group
# EACHI is a newer data table feature
# https://cran.r-project.org/web/packages/data.table/data.table.pdf 
# Advanced: When i is a list (or data.frame or data.table), DT[i, j, by=.EACHI]
# evaluates j for the groups in 'DT' that each row in i joins to. That is, you can
# join (in i) and aggregate (in j) simultaneously. We call this grouping by each i.
# See this StackOverflow answer for a more detailed explanation until we roll out
# vignettes
```

```{r}
dt[ c("A","C"),  sum(v4), by=v2]    # operate by each group, without .EACHI
# ---- I will have to go back to the link and read up on this feature.  When does it help?
```



```{r}
setkey(dt, v1, v2)    # the index order determines the implied columns for later row selection
dt                    # to show the change.   NOTE: R Markdown may just display the first 10 of 12 rows

dt[ .(2,"C")]         # select where (v1 == 2) and (v2 == "C")    # NOTE: last 2 rows were selected.
```

```{r}
dt[ .(2, c("A","C"))]         # select where (v1 == 2) and ((v2 == "A") or (v2 == "C"))   

```


# =========================================================
# ==== Advanced data.table Operations =====================

```{r}
setkey(dt, v1, v2)   # to reset
# .N is an internal data.table variable for the Number of rows
dt[ .N-1 ]   # return the next to last row
dt[, .N ]    # return the number of rows, i.e. 12   same as nrow(dt)
```

```{r}
dt[, .(v2,v3)]     # return a data table with just these two columns.   Select v2, v3 from dt
dt[, list(v2,v3)]  # same as above
```

```{r}
dt   # show for comparison

dt[, mean(v3), by = .(v1,v2)]    # select mean(v3) from dt group by (v1, v2)

```

```{r}
dt[, v5 := mean(v3), by = .(v1,v2)]    # select mean(v3) from dt group by (v1, v2)
dt    # see the new col added to the data table
```


# =========================================================
# ==== .SD & .SDcols ======================================

# .SD is an internal Sub Data frame returned in by-group processing
# the .SD does not include the by-group var

```{r}
dt[, print(.SD), by=v2]
```

```{r}
dt[, .SD[c(1, .N)], by=v2]   # select where the first(1) and last(.N) row of each group by v2
```


```{r}
dt[, lapply(.SD, sum), by = v2]  # use the list apply, over the data frame .SD by v2 group, to calculate a sum
```

```{r}
dt[, lapply(.SD, sum), by = v2,
   .SDcols=c("v3", "v4")]        # same as above, but select 2 columns for output from the group by "Subset Data columns"
```


```{r}
dt[, lapply(.SD, sum), by = v2,
   .SDcols = paste0("v", 3:4)]        # same as above, showing text processing to create the col names selected
```


# =========================================================
# ==== Chaining ===========================================

```{r}
# ---- first, reset the dt
dt <- data.table( v1 = c(1L,2L),  # and call it DT
                  v2 = LETTERS[1:3],
                  v3 = round(rnorm(4),4),
                  v4 = 1:12)
dt  # show before

# ---- after, step by step
dt <- dt[, .(v4.sum = sum(v4)), by = v1]   
dt  # show changes

dt[40 < v4.sum]
```



```{r}
# ---- first, reset the dt
dt <- data.table( v1 = c(1L,2L),  # and call it DT
                  v2 = LETTERS[1:3],
                  v3 = round(rnorm(4),4),
                  v4 = 1:12)
dt  # show before

# ---- after, 1 step
dt <- dt[, .(v4.sum = sum(v4)), by = v1][40 < v4.sum]
dt
```


```{r}
# ---- first, reset the dt
dt <- data.table( v1 = c(1L,2L),  # and call it DT
                  v2 = LETTERS[1:3],
                  v3 = round(rnorm(4),4),
                  v4 = 1:12)
dt  # show before

# ---- after, 1 step (different)
dt <- dt[, .(v4.sum = sum(v4)), by = v1] [order(-v1)]  
dt  # show changes

```


# =========================================================
# ==== set() family =======================================
# ==== set() ==============================================
# syntax:   for (i in from:to)   set(DT, row, column, new value)

```{r}
# ---- first, reset the dt
dt <- data.table( v1 = c(1L,2L),  # and call it DT
                  v2 = LETTERS[1:3],
                  v3 = round(rnorm(4),4),
                  v4 = 1:12)

rows <- list(3:4, 5:6)     # first over rows 3 and 4;   then over rows 5 and 6 
cols <- 1:2                # for columns v1 and v2
rows
cols

for (i in seq_along(rows))
{set(dt,
     i =     rows[[i]],
     j =     cols[i],
     value = NA)}          # for the above iteration, assign NA to the value

dt   # after
```


# =========================================================
# ==== setnames() =========================================
# syntax:  setnames(dt, "old", "new")[]

```{r}
setnames(dt, "v2", "rating")     # rename the column "v2" to "rating"
names(dt)
```


```{r}
# ---- first, reset the dt
dt <- data.table( v1 = c(1L,2L),  # and call it DT
                  v2 = LETTERS[1:3],
                  v3 = round(rnorm(4),4),
                  v4 = 1:12)

setnames(dt, 
         c("v2", "v3"), 
         c("v2.rating", "v3.dc"))

names(dt)
```

# ---- syntax:  setcolorder(dt, "neworder")    set column order
```{r}
setcolorder(dt, c("v4", "v1", "v3.dc", "v2.rating"))
names(dt)
```


# =========================================================
# https://rstudio-pubs-static.s3.amazonaws.com/52230_5ae0d25125b544caab32f75f0360e775.html
# ==== joining data.tables ================================





# =========================================================
# ==== xxxxxxxxxxxxxxxx ===================================




# =====================================================================================================================
