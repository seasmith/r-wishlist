# R Wishlist
Things I wish base R had

## New Functions

* `notNA()` = compliment to `is.na()`
* `allNA()` = are all elements `NA`?
* `noneNA()` = compliment to `allNA()`

```r
# Possible implementations

notNA  <- function (x) !is.na(x)
allNA  <- function (x) all(vapply(x, is.na, logical(1))
noneNA <- function (x) all(complete.cases(x))
```

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

## New Behavior

* `dirname(path, steps = 1)` = how many steps to go back? 
    * `steps` should default to `1` (because that is the current behavior) and be limited by new option `max.steps` (default = `15`)
* `list.files(path, recursive, direction = Inf)` = control the depth and direction of recursion
    * allow files to be listed in both recursively forwards and backwards
    * control depth of recursion (default would be `Inf`)
* `setdiff(x = "some/file/path/here", y = "some/file")` = allow setdiffing on file paths
    * would help determine path distance between two file paths
    * would be difficult to implement -- prefer `file_setdiff()` instead
