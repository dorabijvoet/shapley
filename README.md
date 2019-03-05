
<!-- README.md is generated from README.Rmd. Please edit that file -->

# shapley

[![CRAN
Version](https://www.r-pkg.org/badges/version/shapley)](https://CRAN.R-project.org/package=shapley)
[![Build
Status](https://travis-ci.org/elbersb/shapley.svg?branch=master)](https://travis-ci.org/elbersb/shapley)
[![Coverage
status](https://codecov.io/gh/elbersb/shapley/branch/master/graph/badge.svg)](https://codecov.io/github/elbersb/shapley?branch=master)

The Shapley value is a concept from game theory that quantifies how much
each player contributes to the game outcome (Shapley 1953). The concept,
however, has many more use cases: it provides a method to quantify the
importance of predictors in regression analysis or machine learning
models, and can be used in a wide variety of decomposition problems
(Shorrocks 2013). Most implementations focus on one narrow use case,
although the algorithm for the Shapley value decomposition is always the
same – it is just the concrete value function that varies. This package
provides a simple algorithm for the Shapley value decomposition.

## Installation

``` r
devtools::install_github("elbersb/shapley")
```

## Usage

The package provides a `shapley` function that takes two arguments: the
value function and a vector of factor names. The value function needs to
be an R function that takes one argument. The `shapley` function will
call the value function repeatedly, each time with a different set of
factors.

For a very simple example, consider that an outcome is determined by two
factors “A” and “B”, which contribute 1 and 2, respectively. The factors
are linearly additive, which makes the use of Shapley value
decomposition unnecessary, but this allows us to validate the results.
The value function is thus defined as:

``` r
simple <- function(factors = c()) {
    value <- 0
    if ("A" %in% factors) value <- value + 1
    if ("B" %in% factors) value <- value + 2
    return(value)
}
```

We now supply the value function to `shapley`, along with the factor
names:

``` r
shapley(simple, c("A", "B"), silent = TRUE)
#>   factor value
#> 1      A     1
#> 2      B     2
```

As expected, the marginal contributions of the two factors are 1 and 2,
respectively. For factor “A”, the underlying computation
is:

``` r
1/2 * (simple("A") - simple()) + 1/2 * (simple(c("A", "B")) - simple("B"))
#> [1] 1
```

### Example 1: Game theory

For this example (taken from
[Wikipedia](https://en.wikipedia.org/wiki/Shapley_value#Glove_game)),
consider three players. Players 1 and 2 supply right-hand gloves, while
Player 3 supplies a left-hand glove. The game is only successful if
players with both types of gloves enter into a coalition. We thus define
the value function as 1 if pairs `{1,3}`, `{2,3}` or `{1,2,3}` are
formed, and 0 otherwise.

In R, we can define the value function as follows:

``` r
glove <- function(factors) {
    if (length(factors) > 1 & 3 %in% factors) return(1)
    return(0)
}
```

To compute the marginal contributions of each player, use:

``` r
shapley(glove, c(1, 2, 3), silent = TRUE)
#>   factor     value
#> 1      1 0.1666667
#> 2      2 0.1666667
#> 3      3 0.6666667
```

### Example 2: Relative importance of predictors

Consider this simple regression model and its R<sup>2</sup>:

``` r
model <- lm(mpg ~ wt + qsec + am, data = mtcars)
summary(model)$r.squared
#> [1] 0.8496636
```

The Shapley value decomposition allows us to determine how much each
predictor contributes to the R<sup>2</sup>. To do this, we need to
define the value function in a way that it runs the regression with the
appropriate subset of predictors. It should return 0 when there are no
predictors:

``` r
reg <- function(factors) {
    if (length(factors) == 0) return(0)
    formula <- paste0("mpg ~ ", paste(factors, collapse = "+"))
    m <- lm(formula, data = mtcars)
    summary(m)$r.squared
}

# test - should be the same as above:
reg(c("wt", "qsec", "am"))
#> [1] 0.8496636
```

``` r
shapley(reg, c("wt", "qsec", "am"), silent = TRUE)
#>   factor     value
#> 1     wt 0.4792448
#> 2   qsec 0.1574791
#> 3     am 0.2129397
```

Note that there are many packages (e.g.,
[relaimpo](https://cran.r-project.org/package=relaimpo)) that provide
this functionality specifically for regression analysis.

## References

Shapley, L. S. (1953). A value for n-person games. Contributions to the
Theory of Games, 2(28), 307-317.

Shorrocks, A. F. (2013). Decomposition procedures for distributional
analysis: a unified framework based on the Shapley value. Journal of
Economic Inequality, 1-28.
