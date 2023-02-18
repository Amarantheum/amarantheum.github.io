---
layout: post
title:  "Week 3 - Math + Resonators"
date:   2023-01-29 00:21:32 -0800
categories: 420-Capstone
mathjax: true
---

## Summary

The focus of this week was the resonator implementation. In order to explain how the resonator works, we're first going to have to dive into some math.

### Goals Completed

- [x] Resonator Implementation
- [x] Resonator Array Implementation

## Mathematical Background
We define the discrete time Fourier transform (DTFT, not to be confused with the discrete Fourier transform) of a signal $x[n]$ as:

$$
X(e^{j \omega}) \overset{\Delta}{=} \sum_{n=-\infty}^{\infty}x[n]e^{-j \omega n}
$$

This gives us a continuous $2\pi$ periodic function of $\omega$ (a frequency between 0 and $2\pi$) that describes the frequency content of the original signal. In the same way, we can define the frequency response of a LTI (linear time invariant) system $H(e^{j \omega})$ as the Fourier transform of its impulse response $h[n]$. In our case, we want to go in the reverse direction and build a system that creates resonances at desired frequencies. To do this, we'll use a new tool, [the Z-transform][z-transform-wiki], to shape the frequency response of our system to achieve the desired results.

The Z-transform is a more generalized form of the DTFT that includes the entire complex plane (just as the Laplace transform is a more generalized version of the continuous time Fourier transform). The Z-transform is defined as:

$$
\mathcal{Z}\{x[n]\} = X(z) \overset{\Delta}{=} \sum_{n=-\infty}^{\infty}x[n]z^{-n}
$$

where $z$ is any value in the complex plane. We can see that the Z-transform generalizes the DTFT by replacing $e^{-j \omega}$ with $z$. This means that in the Z-plane, the DTFT corresponds to the unit circle as shown in the image below.

![image](/assets/images/PoleZeroPlotBlank.svg)

The angle of a point on the unit circle relative to the positive real axis corresponds to a particular frequency. Given a sample rate $s$, we can convert a frequency $f$ in Hz to an angle on the unit circle with the equation:

$$\omega = 2\pi\frac{f}{s}$$

**notice that the Nyquist frequency is at -1 which means any value on the top half the unit circle and its complex conjugate correspond to the same frequency*

To manipulate the frequency response of our system, we can place poles and zeros in the Z-plane in desired locations. By placing poles and zeros, we mean creating a transfer function in the Z-domain that goes to infinity at certain points and goes to zero at other points respectively. For example, the transfer function $X(z) = \frac{z - 1}{z^2+\frac{1}{4}}$ has a zero at $z=1$ and two poles at $z=\pm \frac{j}{2}$ which corresponds to the following pole-zero plot:

![image](/assets/images/PoleZeroPlotExample.svg)

This would indicate that frequencies on the unit circle near 0 will be attenuated while frequencies on the unit circle near $\pm \frac{j}{2}$ will be amplified.

## Resonator Implementation

We can make use of the properties of poles in the Z-plane to create an LTI system that amplifies specific frequencies by placing poles arbitrarily close to the unit circle at the desired frequencies (the pole at 0 will be important later but does not affect the magnitude of the frequency response):

![image](/assets/images/PoleZeroPlotResonance.svg)

This results in a system with the following frequency response:

![image](/assets/images/pole_zero_freq_resp.png)

We will now derive the equation for the generalized resonator:

Given a target frequency $\omega$ and an intensity value $r \in [0, 1)$, we can write the transfer function of our system as:

$$ H(z) = \frac{z^{-1}}{(1-re^{j\omega}z^{-1})(1-re^{-j\omega}z^{-1})} $$

This places a pole at 0 and two poles at $re^{\pm j \omega}$. The reason for having conjugate poles is to keep the output of the system real-valued, and, as shown previously, conjugate points on the unit circle correspond to the same frequency due to Nyquist theory.

$$
\begin{align*}
\frac{z^{-1}}{(1-re^{j\omega}z^{-1})(1-re^{-j\omega}z^{-1})} &= \frac{z^{-1}}{1-(re^{j\omega} + re^{-j\omega})z^{-1}+r^2z^{-2}}\\
&= \frac{z^{-1}}{1-2r\cos(\omega)z^{-1}+r^2z^{-2}}\\
\end{align*}
$$

using the properties of the inverse Z-transform, we can transform this into the following difference equation:

$$
\begin{align*}
\implies y[n] - 2r\cos(\omega)y[n-1]+r^2y[n-2] &= x[n-1]\\
\implies y[n] &= x[n-1] + 2r\cos(\omega)y[n-1] - r^2y[n-2]\\
&= x[n-1] + 2r\cos(2\pi\frac{f}{s})y[n-1] - r^2y[n-2]
\end{align*}
$$

This is the generalized difference equation for a resonant filter given an arbitrary frequency and sample rate where $x[n]$ is the incoming signal and $y[n]$ is the signal after passing through our LTI system. As you can see, this is quite easy to implement programmatically and, more importantly, is **computationally efficient** to the point that we can process a signal with many resonators in parallel.

There are many resources online for understanding the final conversion from transfer function to difference equation but I can update this post if someone requests further explanation.

[gp-resonator-docs]: https://amarantheum.github.io/GPResonatorLibrary 
[z-transform-wiki]: https://en.wikipedia.org/wiki/Z-transform