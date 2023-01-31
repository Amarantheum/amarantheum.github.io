---
layout: post
title:  "Week 3 - Math + Resonators"
date:   2023-01-29 00:21:32 -0800
categories: 420 Capstone
---

The focus of this week was the resonator implementation.
In order to explain how the resonator works, we're first going to have to dive into some math.

We define the discrete time Fourier transform (not to be confused with the discrete Fourier transform)
of a signal $x[n]$ as:
$$
\begin{equation}
X(e^{jw})=\sum_{n=-\infin}^{\infin}x[n]e^{-jwn}
\end{equation}
$$

![image](/assets/images/PoleZeroPlotBlank.svg)

Above is an image of the unit circle in the Z-plane (complex plane). When we take the
discrete time Fourier transform of a signal,


[gp-resonator-docs]: https://amarantheum.github.io/GPResonatorLibrary 