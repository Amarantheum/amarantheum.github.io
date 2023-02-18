---
layout: post
title:  "Non-Linear Regression for Resonant Filters"
date:   2023-02-17 00:16:37 -0800
categories: 420 Capstone
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

[Least squares][least-squares] is an approach to this problem that uses the sum of the squares of the residuals of the model as a function that we must minimize. Residuals are the distance between each point and the corresponding model prediction for that point.

![image](/assets/images/non-linear-regression/residual-ex.png)

In other words, we want to minimize the distance between our model prediction and each of the points so that our model better predicts the data. This function can be written as:

$$f(M)=\sum_{(x, y) \in P}\frac{1}{2}(M(x) - y)^2$$

where $$M$$ is our model and $$M(x)$$ is the prediction of our model for input $x$.

[least-squares]: https://en.wikipedia.org/wiki/Least_squares