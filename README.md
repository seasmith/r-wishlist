# R Wishlist
Things I wish base R could have

## Table of Contents
* [New Functions](#new-functions)
  * [Assignment](#assignment)
  * [Vectors](#vectors)
  * [Numbers](#numbers)
  * [Logicals](#logicals)
  * [Dates And Time](#dates-and-time)
  * [Files](#files)
  * [Environments](#environments)
* [New Behavior](#new-behavior)
* [Argument Order](#argument-order)
* [Remove](#remove)

## New Functions

### Assignment

* `:=` = deconstruction operator
* `<->` = compound assignment

```r
# := is a valid but undefined infix operator.
# This implementation does not use a base R solution!

`:=` <- zeallot::`%<-%`

# I have only heard rumors of _'s validity,
# but have yet to see this applied.
# So, there is no way to implement a compound
# assignment operator with _.

# I chose <-> becuase I currently like it
# but thought mentioning _ was good.
```

### Vectors

These functions apply to (just about) any vector.

* `n_duplicated()` = the number of duplicated elements (elements which are duplicates of some other element)

```r
n_duplicated <- function (x, na.rm = FALSE, incomparables = FALSE, ...) {
    
    x <- if (na.rm) c(na.omit(x)) else x
    sum(duplicated(x, incomparables = incomparables, ...))
    
}
```

### Numbers

* `rep_each()` = the `each` argument in `rep()` is not obvious from the docs; we already have `rep_len()`; see also  `rep()` in [new behavior](#new-behavior)

```r
# Possible implementations

rep_each <- function (x, each) rep(x, each = each)

# Faster, from: https://twitter.com/BrodieGaslam/status/1206765767775203328
rep_each <- function(x, each) {
  if(each) {
    res <- matrix(x, nrow=each, ncol=length(x), byrow=TRUE)
    dim(res) <- NULL
    res
  } else x[0]
}
```

* `rm_na()` = remove NA placeholders in a vector; `na.omit()` returns a new class and attribute that is undesired in some context (i.e. printing).

```r
# Possible implementation

rm_na <- function(x) c(na.omit(object = x))
```

* `index()` -- index or normalize a vector

```r
index <- function (x, method = "min-max") 
{
    x <- switch(method,
      `min-max` = {
        min_x <- min(x, na.rm = TRUE)
        max_x <- max(x, na.rm = TRUE)
        vapply(X = x,
               FUN = function(i) (i - min_x)/(max_x - min_x),
               FUN.VALUE = numeric(1))},
      start = {
        first_x <- x[1]  # assume first is start
        vapply(X = x,
               FUN = function(i) (i - first_x)/first_x, 
               FUN.VALUE = numeric(1))
      })
    x
}
```

* `cumpct()` -- cumulative percent (ascending - from smallest to largest)

```r
cumpct <- function (x) {
  i <- order(x)
  j <- order(i)
  xi <- x[i]
  res <- cumsum(xi) / sum(xi)
  res[j]
}

d <- c(25, 10, 50, 15)
cumpct(d)
# [1] 0.50 0.10 1.00 0.25
```

### Logicals

* `notNA()` = compliment to `is.na()`
* `allNA()` = are all elements `NA`?
* `noneNA()` = compliment to `allNA()`

```r
# Possible implementations

notNA  <- function (x) !is.na(x)
allNA  <- function (x) all(is.na(x) == TRUE)
noneNA <- function (x) all(is.na(x) == FALSE)
```

* `none()` = compliment to `all()`
* `allFALSE()` = opposite of `all()`
* `nor()` = neither are `TRUE`; binary case of `!all()`/`none()`

```r
# Possible implementations

none <- function(...) all(... != TRUE)
allFALSE <- function(...) all(... == FALSE)
nor <- function (x, y) all(x != TRUE, y != TRUE)
```

### Dates And Time

* `is.POSIXct()` = datetime equivalent to `is.Date()`
* `is.POSIXlt()` = ...
* `is.POSIXt()` = ...

```r
# Possible implementations

is.POSIXct <- function (x) inherits(x, "POSIXct")
is.POSIXlt <- function (x) inherits(x, "POSIXlt")
is.POSIXt <- function (x) inherits(x, "POSIXt")
```

### Lists

* `list_collapse(x, index)` -- collapse a list's components by an index vector

```r
# Possible implementations

l <- rep(list(1:3, 4:5), 2)

# -- Using split + lapply -- #
list_collapse <- function (x, index) {
    lapply(split(x, index), function (j) Reduce(append, j))
}

collapse_index1 <- rep(1:2, times = 2)
collapse_index2 <- rep(1:2, each = 2)

list_collapse(l, collapse_index1)
list_collapse(l, collapse_index2)

# -- Using tapply -- #

list_collapse2 <- function (x, index) tapply(x, index, function (j) Reduce(append, j))

list_collapse2(l, collapse_index1)
list_collapse2(l, collapse_index2)

# -- Inner function -- #
# The function function (j) Reduce(append, j) 
# is the real workhorse. Can test against using
# c in place of append. Also test list_collapse
# against list_collapse2.
# 
# However, I have discovered that these are 
# merely curious re-discoveries of unlist().

list_collapse3 <- function (x, index) tapply(x, index, unlist)

list_collapse3(l, collapse_index1)
list_collapse3(l, collapse_index2)

# Alternatively, could allow conditional input
# rather than an index vector.
```

### Files

* `find_files(files, steps = 1L, path = ".")` = find a vector of file names; would be useful when looking for an `.Rproj` file to call home

```r
# Possible implementations

# This function takes no file names as input
# but it behaves similar to how find_files()
# would behave.
# Need to add a stop for the number of iterations.
find_rproj <- function(wd = getwd(), it = 1L) {
    found <- length(list.files(wd, pattern = "\\.Rproj")) == 1
    if (found) {  # Found it.
        return(wd) 
    } else if (wd == "." | grepl("\\:/", wd)) { # Stop if nothing found.
        warning("Could not find Rproj file.")
        return(invisible(NULL))
    } else if (it > 5L ) {  # Stop before things get out of control.
        warning("Could not find Rproj file.")
        return(invisible(NULL))
    } else {
        it <- it + 1L
        find_rproj(wd <- dirname(wd)) # Keep going
    }
}
```

* `file_setdiff(x, y)` = path distance between two file paths
    * possible spin-offs
        * `file_intersect(x, y)` = path in common between two file paths (hmmm)
        * `file_union(x, y)` = combined distinct file path (hmmm)

```r
# Expected behavior

file_setdiff("some/file/path/here", "some/file")
#> [1] "path/here"

file_setdiff("some/file/path", "another/file/path")
#> [1] character(0)
```

### Environments

* `lock()` = lock object in current environment (taken from [here](https://colinfay.me/js-const-r/))

```r
lock <- function(x){
  lockBinding(
    deparse(
      substitute(x)), 
    env = parent.frame()
  )
}

plop <- 12
lock(plop)
plop <- 13
```

___

## New Behavior

* `rep(x, ..., sort = FALSE)` = sort the repitions; who can remember `each`?
* `dirname(path, steps = 1)` = how many steps to go back? 
    * `steps` should default to `1` (because that is the current behavior) and be limited by new option `max.steps` (default = `15`)
* `list.files(path, recursive, direction = Inf)` = control the depth and direction of recursion
    * allow files to be listed in both recursively forwards and backwards
    * control depth of recursion (default would be `Inf`)
* `setdiff(x = "some/file/path/here", y = "some/file")` = allow setdiffing on file paths
    * would help determine path distance between two file paths
    * would be difficult to implement -- prefer `file_setdiff()` instead

___

## Argument Order

New argument order of existing functions

1. `x` should ALWAYS come first
* `Filter(x, f)`
* `Reduce(x, f, init, accumulate = FALSE, right = FALSE)`

2. `pattern` should follow `x`; `ignore.case` and `fixed` are more relevant than `perl`; also, see # 1
* `grep(x, pattern, ignore.case = FALSE, value = FALSE, fixed = FALSE, invert = FALSE, useBytes = FALSE, perl = FALSE)`
* `grepl(x, pattern, ignore.case = FALSE, fixed = FALSE, useBytes = FALSE, perl = FALSE)`
* `gsub(x, pattern, replacement, ignore.case = FALSE, fixed = FALSE, useBytes = FALSE, perl = FALSE)

___

## Remove

Remove a function from base R

* `as.name()` --OR-- `as.symbol()` = they are the exact same thing (`identical(as.name, as.symbol) # [1] TRUE`); we only need one
