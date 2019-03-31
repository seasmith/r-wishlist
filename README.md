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

* `find_files(files, steps = 1L, path = ".")` = find a vector of file names; would be useful when looking for an `.Rproj` file to call home

```r
# Possible implementations

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

## New Behavior

* `dirname(path, steps = 1)` = how many steps to go back? `steps` should default to `1` (because that is the current behavior) and be limited by new option `max.steps` (default = `15`)
* `list.files(path, recursive, direction = Inf)` = control the depth and direction of recursion

