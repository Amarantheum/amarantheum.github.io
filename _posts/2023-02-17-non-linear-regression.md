---
layout: post
title:  "Non-Linear Regression for Resonant Filters"
date:   2023-02-17 00:16:37 -0800
categories: modal-synthesis
mathjax: true
---

## Summary

The focus for these past few weeks was on curve fitting using non-linear regression for the purpose of fitting filters to audio spectrums. The documentation for the library can be found [here](https://amarantheum.github.io/GPResonatorLibrary){:target="_blank"}.

### Goals Completed

- [x] Implement non-linear regression for frequency spectrum curves

## Regression and Least Squares

![image](/assets/images/non-linear-regression/regression-ex.png)

Regression is the process of taking a set of points in a space with both an independent and dependent variable and fitting an equation (model) to describe the relationship between the two.

**Note that the dependent variable can be multi-dimensional*

[Least squares](https://en.wikipedia.org/wiki/Least_squares){:target="_blank"} is an approach to this problem that uses the sum of the squares of the residuals of the model as an objective function that we must minimize. Residuals are the distance between each point and the corresponding model prediction for that point.

![image](/assets/images/non-linear-regression/residual-ex.png)

In other words, we want to minimize the distance between our model prediction and each of the points so that our model better predicts the data. The objective function can be written as:

$$
\begin{align*}
f(M)&=\sum_{(x, y) \in P}\frac{1}{2}(M(x) - y)^2\\
&=\sum_{(x, y) \in P}\frac{1}{2}(\hat{y} - y)^2
\end{align*}
$$

where $P$ is the set of points, $M$ is our model, and $M(x)=\hat{y}$ is the prediction of our model for input $x$. In many cases, we can solve this equation by calculating the [gradient vector](https://en.wikipedia.org/wiki/Gradient){:target="blank_"}
of the objective function and setting the gradient vector to 0. In other cases, the equation is minimized using a form of [gradient descent](https://en.wikipedia.org/wiki/Gradient_descent){:target="blank_"} which is an iterative process that minimizes the objective function at each iteration.

## Resonant Filter Fitting

Using this theory, we can create a model using our resonant filter definition and try to fit it to a set of points that we believe represents a resonance.

For example, given the following set of points representing the magnitude of the frequency of a signal:

![image](/assets/images/non-linear-regression/scatter.png)

we can attempt to fit the following resonance curve:
![image](/assets/images/non-linear-regression/curve-fit-ex.png)

## Formal Mathematical Methodology

The resonance equation we derived in the [previous post][previous-post] will be the basis for our model:

$$
H(z) = \frac{r\sin(\omega_0)z^{-1}}{(1-re^{j\omega_0}z^{-1})(1-re^{-j\omega_0}z^{-1})}
$$

In our case, we are interested in the magnitude of the frequency response of the filter which corresponds to values of $z$ on the unit circle in the complex plane. First we multiply the numerator and denominator by $z^2$ for simplicity:

$$
\begin{align*}
H(z) &= \frac{r\sin(\omega_0)z}{(z-re^{j\omega_0})(z-re^{-j\omega_0})}\\
\implies |H(e^{j\omega})|&=\left| \frac{r\sin(\omega_0)e^{j\omega}}{(e^{j\omega}-re^{j\omega_0})(e^{j\omega}-re^{-j\omega_0})} \right|\\
&=\frac{|r\sin(\omega_0)e^{j\omega}|}{|e^{2j\omega}-2r\cos(\omega_0)e^{jw}+r^2|}\\
&=\frac{r\sin(\omega_0)}{|\cos(2\omega)+j\sin(2\omega)-2r\cos(\omega_0)(\cos(\omega)+j\sin(\omega))+r^2|}\\
&=\frac{r\sin(\omega_0)}{\sqrt{(\cos(2\omega)-2r\cos(\omega_0)\cos(\omega)+r^2)^2+(\sin(2\omega)-2r\cos(\omega_0)\sin(\omega))^2}}
\end{align*}
$$

This gives us the magnitude of the frequency response of a resonant filter in terms of:
 - $r$ - the magnitude of the conjugate poles
 - $\omega$ - the frequency (independent variable)
 - $\omega_0$ - the center frequency of the poles

In addition, we can add an extra gain variable that mimics our ability to amplify and attenuate the output of each of our resonators in the resonator array. We'll also assume that $\omega_0$ is given to use during the peak finding algorithm and is therefore known before computation. This gives us the following equation to perform regression with:

$$
\begin{align*}
f(g, r)&=\frac{g\cdot r\sin(\omega_0)}{\sqrt{(\cos(2\omega)-2r\cos(\omega_0)\cos(\omega)+r^2)^2+(\sin(2\omega)-2r\cos(\omega_0)\sin(\omega))^2}}\\
\implies R(g, r) &= f(g,r)-y
\end{align*}
$$

At this point, we could find the gradient $\nabla R$ of the residual function and perform gradient descent to find optimal $g$ and $r$ values to match a given set of points $(\omega, y)$. However, the current residual function has an ugly, discontinuous, non-convex equation which may not be clean to work with. In order to make this easier to differentiate, the residual equation $R$ is rewritten by manipulating both sides of the equation:

$$
\begin{align*}
y&=\frac{g\cdot r\sin(\omega_0)}{\sqrt{(\cos(2\omega)-2r\cos(\omega_0)\cos(\omega)+r^2)^2+(\sin(2\omega)-2r\cos(\omega_0)\sin(\omega))^2}}\\
 \implies g^2r^2\sin^2(\omega_0) &= y^2\left[(\cos(2\omega)-2r\cos(\omega_0)\cos(\omega)+r^2)^2+(\sin(2\omega)-2r\cos(\omega_0)\sin(\omega))^2\right]\\
 \implies 0 &= g^2r^2\sin^2(\omega_0) - y^2\left[(\cos(2\omega)-2r\cos(\omega_0)\cos(\omega)+r^2)^2+(\sin(2\omega)-2r\cos(\omega_0)\sin(\omega))^2\right]\\
 \implies R(g, r) &= g^2r^2\sin^2(\omega_0) - y^2\left[(\cos(2\omega)-2r\cos(\omega_0)\cos(\omega)+r^2)^2+(\sin(2\omega)-2r\cos(\omega_0)\sin(\omega))^2\right]
\end{align*}
$$

The residual equation is now is a nicer form that is continuous with respect to all variables. and we just need to find the gradient:

$$
\nabla R(g, r)=\begin{bmatrix}
2g r^2 \sin^2(\omega_0)\\
2g^2r\sin^2(\omega_0) - 4 y^2  (r - \cos(\omega_0) \cos(\omega)) (r (r - 2\cos(\omega_0)\cos(\omega)) + \cos(2\omega)) + 8 y^2 \cos(\omega_0) \sin^2(\omega) (\cos(\omega) - r \cos(\omega_0))
\end{bmatrix}
$$

With this, we have everything that is necessary to find $g$ and $r$ for our resonator given a set of points in the frequency domain (we simply use gradient descent to minimize the square of all residuals). The next step will be to analyze frequency spectrums to determine the locations of peaks.

[previous-post]: https://amarantheum.github.io/modal-synthesis/2023/02/10/decay.html