---
layout: post
title:  "Non-Linear Regression for Resonant Filters"
date:   2023-02-17 00:16:37 -0800
categories: 420-Capstone
mathjax: true
---

## Summary

The focus for these past few weeks was on curve fitting using non-linear regression for the purpose of fitting filters to audio spectrums.

### Goals Completed

- [x] Implement non-linear regression for frequency spectrum curves

## Regression and Least Squares

![image](/assets/images/non-linear-regression/regression-ex.png)

Regression is the process of taking a set of points in a space with both an independent and dependent variable and fitting an equation (model) to describe the relationship between the two.

**Note that the dependent variable can be multi-dimensional*

[Least squares](https://en.wikipedia.org/wiki/Least_squares){:target="_blank"} is an approach to this problem that uses the sum of the squares of the residuals of the model as an objective function that we must minimize. Residuals are the distance between each point and the corresponding model prediction for that point.

![image](/assets/images/non-linear-regression/residual-ex.png)

In other words, we want to minimize the distance between our model prediction and each of the points so that our model better predicts the data. The objective function can be written as:

$$f(M)=\sum_{(x, y) \in P}\frac{1}{2}(M(x) - y)^2$$

where $P$ is the set of points, $M$ is our model, and $M(x)$ is the prediction of our model for input $x$. In many cases, we can solve this equation by calculating the [gradient vector][gradient]
of the objective function and setting the gradient vector to 0. In other cases, the equation is minimized using a form of [gradient descent][gradient-descent] which is an iterative process that minimizes the objective function at each iteration.

## Resonant Filter Fitting

Using this theory, we can create a model using our resonant filter definition and try to fit it to a set of points that we believe represents a resonance.

For example, given the following set of points representing the magnitude of the frequency of a signal:

![image](/assets/images/non-linear-regression/scatter.png)

we can attempt to fit the following resonance curve:
![image](/assets/images/non-linear-regression/curve-fit-ex.png)

## Formal Mathematical Method

The equation


[previous-post]: https://amarantheum.github.io/420-capstone/2023/02/10/decay.html
[least-squares]: https://en.wikipedia.org/wiki/Least_squares
[gradient]: https://en.wikipedia.org/wiki/Gradient
[gradient-descent]: https://en.wikipedia.org/wiki/Gradient_descent