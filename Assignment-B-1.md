Assignment-B-1
================

``` r
# Load required packages
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

## Exercise 1-2: Creating a Custom Summarize Function

In this exercise, I’ll create a function to calculate a summary
statistic for each group within a data frame. This is useful when we
want to reuse a particular summarizing workflow.

### Step 1: Define the Function (with tags)

This function groups a data frame by a specified column and calculates a
specified summary statistic (either mean or median) and count for
another column within each group.

``` r
#This function groups a data frame by a specified column and calculates a specified 
#' summary statistic (either mean or median) and count for another column within each group.
#'
#' @param data A data frame to perform the grouping and summarizing operation on.
#'   Named "data" to clearly indicate that this is the primary input dataset.
#' @param group_col The column used to group the data, chosen to be flexible in grouping options.
#'   Named "group_col" to signify its role as the grouping variable.
#' @param summary_col The column for which the summary statistic is calculated.
#'   Named "summary_col" to reflect its purpose as the variable being summarized.
#' @param statistic A character string specifying the summary statistic to use, either `"mean"` or `"median"`.
#'   Named "statistic" to denote the summary measure type, allowing flexibility in calculations.
#' @return A tibble with the specified summary statistic and count of the `summary_col` for each unique 
#'   value in `group_col`.
#'   
# Define a flexible summary function with error handling and choice of summary statistic
summarize_by_group <- function(data, group_col, summary_col, statistic = "mean") {
  # Check if summary_col is numeric
  if (!is.numeric(data %>% pull({{ summary_col }}))) {
    stop("The summary column must be numeric.")
  }
  
  # Check if statistic argument is valid
  if (!statistic %in% c("mean", "median")) {
    stop("The 'statistic' argument must be either 'mean' or 'median'.")
  }
  
  # Apply the chosen statistic
  data %>%
    group_by({{ group_col }}) %>%
    summarise(
      summary_value = if (statistic == "mean") mean({{ summary_col }}, na.rm = TRUE) else median({{ summary_col }}, na.rm = TRUE),
      count = n()
    )
}
```

The function is set to handle NA values using na.rm = TRUE. Consistency
in output type is maintained by using summarise() from dplyr, which will
always return a tibble.

## Exercise 3: Demonstrate

### Example 1: Calculating Mean by Group

In this example, I’ll use `summarize_by_group` to calculate the mean
`mpg` for each `cyl` group in the `mtcars` dataset.

``` r
# Calculating mean mpg by cylinder group in the mtcars dataset
summarize_by_group(mtcars, cyl, mpg, statistic = "mean")
```

    ## # A tibble: 3 × 3
    ##     cyl summary_value count
    ##   <dbl>         <dbl> <int>
    ## 1     4          26.7    11
    ## 2     6          19.7     7
    ## 3     8          15.1    14

or directly use:

``` r
summarize_by_group(mtcars, cyl, mpg)
```

    ## # A tibble: 3 × 3
    ##     cyl summary_value count
    ##   <dbl>         <dbl> <int>
    ## 1     4          26.7    11
    ## 2     6          19.7     7
    ## 3     8          15.1    14

Next, we’ll calculate the median mpg by cyl instead. This is helpful to
see how the function adapts to different summary statistics.

``` r
# Calculating median mpg by cylinder group in the mtcars dataset
summarize_by_group(mtcars, cyl, mpg, statistic = "median")
```

    ## # A tibble: 3 × 3
    ##     cyl summary_value count
    ##   <dbl>         <dbl> <int>
    ## 1     4          26      11
    ## 2     6          19.7     7
    ## 3     8          15.2    14

## Exercise 4: Testing

These tests will check the function’s behavior under different scenarios
and ensure it performs as expected.

``` r
# Load the testthat package
library(testthat)
```

    ## Warning: package 'testthat' was built under R version 4.1.2

    ## 
    ## Attaching package: 'testthat'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     matches

``` r
test_that("summarize_by_group works as expected", {
  
  # Test 1: Check if function returns a tibble with correct columns when using mean
  result1 <- summarize_by_group(mtcars, cyl, mpg, statistic = "mean")
  expect_s3_class(result1, "tbl_df")  # Check if output is a tibble
  expect_named(result1, c("cyl", "summary_value", "count"))  # Check for correct column names
  
  # Test 2: Check that the function handles NA values correctly when calculating the mean
  mtcars_with_na <- mtcars
  mtcars_with_na$mpg[1:3] <- NA  # Introduce NA values in mpg
  result2 <- summarize_by_group(mtcars_with_na, cyl, mpg, statistic = "mean")
  expect_true(all(!is.na(result2$summary_value)))  # summary_value should have no NAs due to na.rm = TRUE
  
  # Test 3: Verify that the function calculates the median correctly
  result3 <- summarize_by_group(mtcars, cyl, mpg, statistic = "median")
  expected_medians <- aggregate(mpg ~ cyl, mtcars, median)$mpg
  expect_equal(result3$summary_value, expected_medians, tolerance = 1e-8)  # Compare medians

  # Test 4: Check if the function throws an error when an unsupported statistic is used
  expect_error(summarize_by_group(mtcars, cyl, mpg, statistic = "sum"), "The 'statistic' argument must be either 'mean' or 'median'")
})
```

    ## Test passed 😀
