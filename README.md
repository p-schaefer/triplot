
# triplot

<!-- badges: start -->

[![CRAN\_Status\_Badge](https://www.r-pkg.org/badges/version/triplot)](https://cran.r-project.org/package=triplot)
[![R build
status](https://github.com/ModelOriented/triplot/workflows/R-CMD-check/badge.svg)](https://github.com/ModelOriented/triplot/actions?query=workflow%3AR-CMD-check)
[![Codecov test
coverage](https://codecov.io/gh/ModelOriented/triplot/branch/master/graph/badge.svg)](https://codecov.io/gh/ModelOriented/triplot?branch=master)
[![DrWhy-eXtrAI](https://img.shields.io/badge/DrWhy-eXtrAI-4378bf)](http://drwhy.ai/#eXtraAI)
<!-- badges: end -->

## Overview

The `triplot` package provides tools for exploration of machine learning
predictive models. It contains an instance-level explainer called
`predict_aspects` (AKA `aspects_importance`), that is able to explain
the contribution of the whole groups of explanatory variables.
Furthermore, package delivers functionality called `triplot` - it
illustrates how the importance of aspects (group of predictors) change
depending on the size of aspects.

Key functions:

  - `predict_aspects()` for calculating the feature groups importance
    (called aspects importance) for a selected observation,
  - `predict_triplot()` and `model_triplot()` for summary of automatic
    aspect importance grouping,
  - `group_variables()` for correlated numeric features into aspects.

The `triplot` package is a part of [DrWhy.AI](http://DrWhy.AI) universe.
More information about analysis of machine learning models can be found
in the [Explanatory Model Analysis. Explore, Explain and Examine
Predictive Models](https://pbiecek.github.io/ema/) e-book.

## Installation

``` r
devtools::install_github("ModelOriented/triplot")
```

## Overview (triplot)

`triplot` shows, in one place:

  - the importance of every single feature,
  - hierarchical aspects importance,
  - order of grouping features into aspects.

We can use it to investigate the **instance level** importance of
features (using `predict_aspects()` function) or to illustrate the
**model level** importance of features (using `model_parts()` function
from DALEX package). `triplot` can be only used on numerical features.
More information about this functionality can be found in [triplot
overview](https://modeloriented.github.io/triplot/articles/vignette_aspect_importance.html#hierarchical-aspects-importance-1).

### Basic example (triplot for model)

To showcase `triplot`, we will choose `apartments` dataset from DALEX,
use it’s numeric features to build a model, create DALEX
[explainer](https://modeloriented.github.io/DALEX/reference/explain.html),
`model_triplot()` to calculate `triplot` object and then plot it.

<details>

<summary>Importing dataset and building a model</summary>

``` r
library(DALEX)

apartments_num <- apartments[,unlist(lapply(apartments, is.numeric))]

model_apartments <- lm(m2.price ~ ., data = apartments_num)
```

</details>

``` r
explain_apartments <- DALEX::explain(model = model_apartments, 
                              data = apartments_num[, -1],
                              y = apartments_num[, 1],
                              verbose = FALSE)

tri_apartments <- model_triplot(explain_apartments)

plot(tri_apartments)
```

<img src="man/figures/unnamed-chunk-4-1.png" style="display: block; margin: auto;" />

We can observe that, at the model level, `surface` and `floor` have the
biggest contribution. `Number of rooms` and `surface` are strongly
correlated and together have strong influence on the
prediction.`Construction year` has small influence on the prediction, is
not correlated with `number of rooms` nor `surface` variables. Adding
`construction year` to them, only slightly increases the importance of
this group.

### Basic example (triplot for observation)

Afterwards, we are building triplot for single instance and it’s
prediction.

``` r
new_observation_apartments <- apartments_num[6,-1]

tri_apartments <- predict_triplot(explain_apartments, 
                                  new_observation = new_observation_apartments)

plot(tri_apartments, add_last_group = FALSE)
```

<img src="man/figures/unnamed-chunk-5-1.png" style="display: block; margin: auto;" />

We can observe that for the given apartment `surface` has also
significant, positive influence on the prediction. Adding `number of
rooms`, increases its contribution. However, adding `construction year`
to those two features, decreases the group importance.

We can notice that `floor` has the small influence on the prediction of
this observation, unlike in the model wise analysis.

## Basic example (predict\_aspects)

For this example we use `titanic` dataset with a logistic regression
model that predicts passenger survival. Features are combined into
thematic aspects.

<details>

<summary>Importing dataset and building a model</summary>

``` r
titanic <- DALEX::titanic_imputed

model_titanic_glm <- glm(survived == 1 ~ ., titanic, family = "binomial")

aspects_titanic <-
  list(
    wealth = c("class", "fare"),
    family = c("sibsp", "parch"),
    personal = c("age", "gender"),
    embarked = "embarked"
  )
  (chosen_passenger <- titanic[2,])
```

    ##   gender age class    embarked  fare sibsp parch survived
    ## 2   male  13   3rd Southampton 20.05     0     2        0

</details>

We are interested in explaining the model prediction for the
`chosen_passenger`.

``` r
predict(model_titanic_glm, chosen_passenger, type = "response")
```

    ##         2 
    ## 0.1531932

It turns out that the model prediction for this passenger’s survival is
very low. Let’s see which aspects have the biggest influence on it.

We start by building DALEX
[explainer](https://modeloriented.github.io/DALEX/reference/explain.html)
and use it to call `predict_aspects()` function. Afterwards, we print
and plot function results.

``` r
explain_titanic <- DALEX::explain(model_titanic_glm, 
                           data = titanic[, -8],
                           y = titanic$survived == "yes",
                           predict_function = predict,
                           label = "Logistic Regression",
                           verbose = FALSE)

ai_titanic <- predict_aspects(x = explain_titanic, 
                              new_observation = chosen_passenger[,-8],
                              variable_groups = aspects_titanic)

print(ai_titanic, show_features = TRUE)
```

    ##   variable_groups importance     features
    ## 2          wealth   -0.73757  class, fare
    ## 4        personal    0.16740  age, gender
    ## 3          family    0.15002 sibsp, parch
    ## 5        embarked   -0.02766     embarked

``` r
plot(ai_titanic)
```

<img src="man/figures/unnamed-chunk-9-1.png" style="display: block; margin: auto;" />

We can observe that `wealth` variables have the biggest contribution to
the prediction. This contribution is of a negative type. `Personal` and
`Family` variables have positive influence on the prediction, but it is
much smaller. `Embarked` feature has very small, negative contribution
to the prediction.

## Acknowledgments

Work on this package was financially supported by the
