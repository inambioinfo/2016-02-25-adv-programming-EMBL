---
title: "Unit testing"
author: "Laurent Gatto"
---

These exercises were written by Martin Morgan and Laurent Gatto for a
[Bioconductor Developer Day workshop](http://bioconductor.org/help/course-materials/2013/BioC2013/developer-day-debug/).

# Introduction

> Whenever you are templted to type something into a print statement
> or a debugger expression, write it as a test insted -- Martin Fowler

**Why unit testing?**

- Writing code to test code;
- anticipate bugs, in particular for edge cases;
- anticipate disruptive updates;
- document and test observed bugs using specific tests.

Each section provides a function that supposedly works as expected,
but quickly proves to misbehave. The exercise aims at first writing
some dedicated testing functions that will identify the problems and
then update the function so that it passes the specific tests. This
practice is called unit testing and we use the RUnit package for
this.

See the
[Unit Testing How-To](http://bioconductor.org/developers/how-to/unitTesting-guidelines/)
guide for details on unit testing using the
[`RUnit`](http://cran.r-project.org/web/packages/RUnit/index.html)
package. The
[`testthat`](http://cran.r-project.org/web/packages/testthat/) is
another package that provides unit testing infrastructure. Both
packages can conveniently be used to automate unit testing within
package testing. 

# Example

## Subsetting

### Problem

This function should return the elements of `x` that are in `y`.


```r
## Example
isIn <- function(x, y) {
    sel <- match(x, y)
    y[sel]
}

## Expected
x <- sample(LETTERS, 5)
isIn(x, LETTERS)
```

```
## [1] "W" "Q" "F" "A" "U"
```
But


```r
## Bug!
isIn(c(x, "a"), LETTERS)
```

```
## [1] "W" "Q" "F" "A" "U" NA
```

### Solution

Write a unit test that demonstrates the issue


```r
## Unit test:
library("RUnit")
```

```
## Loading required package: methods
```

```r
test_isIn <- function() {
    x <- c("A", "B", "Z")
    checkIdentical(x, isIn(x, LETTERS))
    checkIdentical(x, isIn(c(x, "a"), LETTERS))

}

test_isIn()
```

```
## Error in checkIdentical(x, isIn(c(x, "a"), LETTERS)): FALSE 
## 
```

Update the buggy function until the unit test succeeds


```r
## updated function
isIn <- function(x, y) {
    sel <- x %in% y
    x[sel]
}

test_isIn() ## the bug is fixed and monitored
```

```
## [1] TRUE
```

## The `testthat` syntax

`expect_that(object_or_expression, condition)` with conditions
- equals: `expect_that(1+2,equals(3))` or `expect_equal(1+2,3)`
- gives warning: `expect_that(warning("a")`, `gives_warning())`
- is a: `expect_that(1, is_a("numeric"))` or `expect_is(1,"numeric")`
- is true: `expect_that(2 == 2, is_true())` or `expect_true(2==2)`
- matches: `expect_that("Testing is fun", matches("fun"))` or `expect_match("Testing is fun", "f.n")`
- takes less: `than expect_that(Sys.sleep(1), takes_less_than(3))`

and

```r
test_that("description", {
    a <- foo()
    b <- bar()
    expect_equal(a, b)
})
```

## Interactive unit testing

```r
library("testthat")
test_dir("./unittests/")
test_file("./unittests/test_foo.R")
```

# Exercises

## Character matching

### Problem

What are the exact matches of `x` in `y`?


```r
isExactIn <- function(x, y)
    y[grep(x, y)]

## Expected
isExactIn("a", letters)
```

```
## [1] "a"
```

```r
## Bugs
isExactIn("a", c("abc", letters))
```

```
## [1] "abc" "a"
```

```r
isExactIn(c("a", "z"), c("abc", letters))
```

```
## Warning in grep(x, y): argument 'pattern' has length > 1 and only the first
## element will be used
```

```
## [1] "abc" "a"
```

<!-- ### Solution -->



## If conditions with length > 1

### Problem

If `x` is greater than `y`, we want the difference of their
squares. Otherwise, we want the sum.


```r
ifcond <- function(x, y) {
    if (x > y) {
        ans <- x*x - y*y
    } else {
        ans <- x*x + y*y
    } 
    ans
}

## Expected
ifcond(3, 2)
```

```
## [1] 5
```

```r
ifcond(2, 2)
```

```
## [1] 8
```

```r
ifcond(1, 2)
```

```
## [1] 5
```

```r
## Bug!
ifcond(3:1, c(2, 2, 2))
```

```
## Warning in if (x > y) {: the condition has length > 1 and only the first
## element will be used
```

```
## [1]  5  0 -3
```

<!-- ### Solution -->



## Know your inputs

### Problem

Calculate the euclidean distance between a single point and a set of
other points.


```r
## Example
distances <- function(point, pointVec) {
    x <- point[1]
    y <- point[2]
    xVec <- pointVec[,1]
    yVec <- pointVec[,2]
    sqrt((xVec - x)^2 + (yVec - y)^2)
}

## Expected
x <- rnorm(5)
y <- rnorm(5)

(m <- cbind(x, y))
```

```
##                x           y
## [1,] -1.18367293 -0.33531853
## [2,] -1.11246294  1.16259283
## [3,]  0.04491836 -0.04160902
## [4,] -0.02398646 -0.87637949
## [5,]  0.21436708 -0.91971939
```

```r
(p <- m[1, ])
```

```
##          x          y 
## -1.1836729 -0.3353185
```

```r
distances(p, m)
```

```
## [1] 0.000000 1.499603 1.263211 1.279695 1.515269
```

```r
## Bug!
(dd <- data.frame(x, y))
```

```
##             x           y
## 1 -1.18367293 -0.33531853
## 2 -1.11246294  1.16259283
## 3  0.04491836 -0.04160902
## 4 -0.02398646 -0.87637949
## 5  0.21436708 -0.91971939
```

```r
(q <- dd[1, ])
```

```
##           x          y
## 1 -1.183673 -0.3353185
```

```r
distances(q, dd)
```

```
##   x
## 1 0
```

<!-- ### Solution -->



## Iterate on 0 length

### Problem

Calculate the square root of the absolute value of a set of numbers.


```r
sqrtabs <- function(x) {
    v <- abs(x)
    sapply(1:length(v), function(i) sqrt(v[i]))
}

## Expected
all(sqrtabs(c(-4, 0, 4)) == c(2, 0, 2))
```

```
## [1] TRUE
```

```r
## Bug!
sqrtabs(numeric())
```

```
## [[1]]
## [1] NA
## 
## [[2]]
## numeric(0)
```

<!-- ### Solution -->



# Unit testing in a package 


## In a package

1. Create a directory `./mypackage/tests`.
2. Create the `testthat.R` file

```r
library("testthat")
library("mypackage")
test_check("sequences")
```

3. Create a sub-directory `./mypackage/tests/testthat` and include as
   many unit test files as desired that are named with the `test_`
   prefix and contain unit tests.

4. Suggest the unit testing package in your `DESCRIPTION` file:

```
Suggests: testthat
```

## Example from the `sequences` package

From the `./sequences/tests/testthat/test_sequences.R` file:

### Object creation and validity

We have a fasta file and the corresponding `DnaSeq` object.

1. Let's make sure that the `DnaSeq` instance is valid, as changes in
   the class definition might have altered its validity.

2. Let's verify that `readFasta` regenerates and identical `DnaSeq`
   object given the original fasta file.

```r
test_that("dnaseq validity", {
  data(dnaseq)
  expect_true(validObject(dnaseq))
})

test_that("readFasta", {
  ## loading _valid_ dnaseq
  data(dnaseq)
  ## reading fasta sequence
  f <- dir(system.file("extdata",package="sequences"),pattern="fasta",full.names=TRUE)
  xx <- readFasta(f[1])
  expect_true(all.equal(xx, dnaseq))
})
```

### Multiple implementations

Let's check that the R, C and C++ (via `Rcpp`) give the same result

```r
test_that("ccpp code", {
  gccountr <-
    function(x) tabulate(factor(strsplit(x, "")[[1]]))
  x <- "AACGACTACAGCATACTAC"
  expect_true(identical(gccount(x), gccountr(x)))
  expect_true(identical(gccount2(x), gccountr(x)))
})
```

## Exercise

Choose any data package of your choice and write a unit test that
tests the validity of all the its data. 

Tips 

- To get all the data distributed with a package, use `data(package = "packageName")`


```r
library("pRolocdata")
data(package = "pRolocdata")
```

- To test the validity of an object, use `validObject`




```r
data(andy2011)
validObject(andy2011)
```

```
## [1] TRUE
```

- Using the `testthat` syntax, the actual test for that data set would be 


```r
library("testthat")
expect_true(validObject(andy2011))
```

