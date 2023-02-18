---
layout: post
title:  "Non-Linear Regression for Resonant Filters"
date:   2023-02-17 00:16:37 -0800
categories: 420 Capstone
mathjax: true
---

## Summary

The focus for these past few weeks was on curve fitting using non-linear regression for the purpose of fitting filters to audio spectrums.
In addition, the interface for adding resonators was simplified to use a decay length in seconds instead of the more abstract "intensity" value.

### Goals Completed

- [x] Simplify interface to use decay
- [x] Implement non-linear regression for filter curves

## Transforming Intensity (radius) to Decay

Previously, the interface for adding resonators to the resonator array involved the frequency and the "intensity" where intensity
corresponded to the radius of the poles. From the [previous post][previous-post], it was shown how we can affect the contour of the
magnitude of the frequency response by manipulating the position of the conjuage poles in the Z-plane. In order to convert the radius
into a decay amount, we calculate the impulse response of the filter. This is done by taking the inverse Z-transform of our transfer equation:

$$
\begin{align*}
H(z) &= \frac{z^{-1}}{(1-re^{j\omega}z^{-1})(1-re^{-j\omega}z^{-1})}\\
\implies h[n] &= \mathcal{Z}^{-1}\{H(z)\}=\frac{r^{n-1}}{\sin(w)}\sin(n)u[n]
\end{align*}
$$

First of all, we can normalize the gain of the resonant filter by multiplying
our transfer function by $$r\sin(w)$$.

$$
\begin{align*}
\implies H(z) &= \frac{r\sin(w)z^{-1}}{(1-re^{j\omega}z^{-1})(1-re^{-j\omega}z^{-1})}\\
\implies h[n] &= \mathcal{Z}^{-1}\{H(z)\}=r^n\sin(n)u[n]
\end{align*}
$$

*\*where $$u[n]$$ is the [unit step function][step-function].*

The impulse response is the time domain response of the system to the
[Kronecker delta function][kronecker-delta] and looks something like:

![image](/assets/images/basic-res-ir.png)

We can think of this function as a periodic sine function that is bounded by an envelop function $$r^n$$ (remember that $$r<1$$):

![image](/assets/images/basic-res-ir-env.png)

In order to calculate $$r$$ given a delay, we simply have to calculate at what value $$n$$ the envelope function $$r^n$$ is equal to some percentage of the original value. In this case, the value 0.1 was chosen which gives us the number of samples to reach 10% of the original amplitude. We just need to solve for $$r$$ in terms of $$n$$:

$$
\begin{align*}
r^n&=0.1\\
\implies n &= \log_{r}(0.1)
\end{align*}
$$

[previous-post]: https://amarantheum.github.io/420/capstone/2023/01/29/week-3-resonators.html

[kronecker-delta]: https://en.wikipedia.org/wiki/Kronecker_delta

[step-function]: https://en.wikipedia.org/wiki/Heaviside_step_function