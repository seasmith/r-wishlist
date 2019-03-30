# R Wishlist
Things I wish base R had

## New Functions

* `notNA()` = compliment to `is.na()`
* `allNA()` = are all elements `NA`?
* `noneNA()` = compliment to `allNA()`

```r
# Possible implementations

notNA  <- function (x) !is.na(x)
allNA  <- function (x) all(vapply(x, FUN = function(x) is.na(x), FUN.VALUE = TRUE))
noneNA <- function (x) all(complete.cases(x))
```

## New Behavior

* `dirname(path, steps = 1L)` = how many steps to go back? `steps` should default to `1L` (because that is the current behavior) and be limited by new option `max.steps` (default = `15L`)
