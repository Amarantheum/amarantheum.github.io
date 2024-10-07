---
layout: post
title:  "Variational Methods for Mode Reconstruction From IRs for Modal Synthesis"
date:   2024-04-27 00:07:57 -0800
categories: 420-Capstone
mathjax: true
---

## Explored Methodologies

All methods described were implemented in the Rust Programming Language since this project builds off of a previous project written in Rust.

### Regression Using Approximate Magnitude (Approximate Regression Method)

A resonant filter used in modal synthesis has an impulse response defined as an exponentially decaying sinusoid. Since we are ignoring phase for now, we can write it as:

$$
\begin{align*}
    h[n] &= g r^n \sin(\omega_0n)u[n]
\end{align*}
$$

where $g\in \mathbb{R}^+$ is some scale amount, $r\in (0, 1)$ is the rate of decay, $\omega_0$ is the frequency of the sinusoid, and $u[n]$ is the unit step function. To get the frequency response, we take the z-transform of $h[n]$:

$$
\begin{equation} % Equation 1 (resonant_filter)
    H(z) = \mathcal{Z}\{h[n]\} = \frac{gr\sin(\omega_0)z^{-1}}{(1 - re^{j\omega_0}z^{-1})(1 - re^{-j\omega_0}z^{-1})} = \frac{gr\sin(\omega_0)z^{-1}}{1-2r\cos(\omega_0)z^{-1}+r^2z^{-2}}
\end{equation}
$$

In the frequency domain, $r$ corresponds to the magnitude of the two poles of $H(z)$. The decay rate of the exponential reflects how dampened a particular resonance is. As $r$ approaches 1, the resonance at frequency $\omega$ decays much slower resulting in a stronger resonance. Using the inverse z-transform, we can derive the second order difference equation that can be used to determine the output of the system $y[n]$ to an incoming signal $x[n]$:

$$
\begin{align*}
    y[n]-2r\cos(\omega_0)y[n-1]+r^2y[n-2]&=gr\sin(\omega_0)x[n-1]\\
    \implies y[n]&=gr\sin(\omega_0)x[n-1]+2r\cos(\omega_0)y[n-1]-r^2y[n-2]
\end{align*}
$$

This means that the response of a single filter to an incoming dry signal can be calculated in $O(n)$ where $n$ is the length of the incoming dry signal since each sample can be calculated in constant time.

To simply things, as stated earlier, we will be working with the magnitude of the frequency domain to avoid additional complexity from imaginary numbers. Finding the magnitude of the frequency response requires substituting $z$ with the parametrization of the unit circle in the complex plane $e^{j\omega}$:

$$
\begin{align}
    |H(e^{j\omega})| &= \frac{|gr\sin(\omega_0)e^{-j\omega}|}{|1-2r\cos(\omega_0)e^{-j\omega}+r^2e^{-2j\omega}|} \nonumber\\
    &=\frac{gr\sin(\omega_0)}{\sqrt{(\cos(2\omega)-2r\cos(\omega_0)\cos(\omega)+r^2)^2+(\sin(2\omega)-2r\cos(\omega_0)\sin(\omega))^2}} % Equation 2 (mag_freq_resp)
\end{align}
$$

This equation is quite complicated in terms of $r$ which makes it messy to work with if we are trying to perform regression with gradient descent. To simplify this, the Approximate Regression Method uses the fact that Equation 1 is holomorphic which means we can use the relative distances of poles and zeros in Equation 1 to create a real-valued function that mimics the shape of Equation 2. Equation 1 has poles that moves closer or farther from the unit circle at frequencies corresponding to $\omega_0$ depending on the value of $r$ as seen in the following figure:

![image](/assets/figures/mode_reconstruction/inverted_poles.svg)

We can mimic this behavior on the real number line by placing poles closer or farther to the real number line depending on the value of $r$ as seen in the following figure:

![image](/assets/figures/mode_reconstruction/inverted_poles_approx.svg)

Doing this, we derive the equation:

$$
\begin{equation}
    M(\omega, g, r)= \frac{gr((\omega-\omega_0)^2+1)}{(\omega-\omega_0)^2+(1-r)^2} % Equation 3 (approx_mag_freq)
\end{equation}
$$

The shape of Equation 3 has a very similar shape to Equation 2 on the domain from 0 to $\pi$ which means we can approximate regression with Equation 2 by using Equation 3.

To perform regression, the [Levenberg-Marquardt Algorithm](https://en.wikipedia.org/wiki/Levenberg%E2%80%93Marquardt_algorithm){:target="_blank"} for least squares optimization is used. This means the function being minimized looks like:

$$
\begin{align}
    f(\mathbf{x}, \mathbf{y}) &= \sum_{i=1}^{n} \frac{1}{2}R(x^{(i)}, y^{(i)}, \mathbf{g}, \mathbf{r})^2\\
    R(x,y,\mathbf{g},\mathbf{r}) &= \sum_{j=1}^{m} \left[M(x, g^{(j)}, r^{(j)})\right] - y
\end{align}
$$

For LMA, we only need the derivative of $R$ in terms of $g$ and $r$ which are straight forward to calculate. The overall algorithm takes $O(mn)$ to calculate the Jacobian matrix of the residual functions and each value of $\omega$ in the frequency domain which means it takes $O(mni)$ to run $i$ iterations.

### Approximate Expectation Maximization

Another approach that can potentially be used to convert IRs into modal spaces is expectation maximization where we treat the magnitude of the frequency response of the IR as a probability distribution. For simplicity, a Gaussian Mixture Model is used to estimate the center frequencies of modes along with their relative strength (larger variance indicates a faster decay rate since the mode is wider which can only happen if we decrease the value of $r$). However, there are a few issues with this approach that will be covered in the discussion section.

Instead of using points sampled from a distribution like a typical GMM, the EM algorithm that is implemented uses the magnitude corresponding to each frequency bin as a weight for each responsibility associated with the bin. This gives us the following EM equations:

$$
\begin{align*}
    r_j^{(i)} &= p(z=j | x^{(i)}) = \frac{\pi_j \mathcal{N}\left(x^{(i)}| \mu_j, \sigma_j^2\right)}{\sum_{j'=1}^m \pi_{j'} \mathcal{N}\left(x^{(i)}| \mu_{j'}, \sigma_{j'}^2\right)}\\
    \pi_j &= \frac{1}{\sum_{i=1}^{n}y^{(i)}}\sum_{i=1}^{n}y^{(i)} r_j^{(i)}\\
    \mu_j&=\frac{1}{\sum_{i=1}^{n}y^{(i)} r_j^{(i)}}\sum_{i=1}^n y^{(i)} r_j^{(i)} x^{(i)}\\
    \sigma_j^2&=\frac{1}{\sum_{i=1}^{n}y^{(i)} r_j^{(i)}}\sum_{i=1}^n y^{(i)} r_j^{(i)} (x^{(i)} - \mu_j)^2
\end{align*}
$$

The implementation uses dynamic programming to prevent repeated calculations which means that each iteration of EM only takes $O(mn)$ since there are $n$ weighted responsibilities for $m$ distributions. The total run time of the algorithm is therefore $O(mni)$ where $i$ is the number of iterations of EM.

One of the main potential drawbacks of EM is that we may not converge to a good minimum if we have a poor initialization. According to [Chen](https://arxiv.org/abs/1305.0626){:target="_blank"}, we can improve the chances of converging to a good minimum by initializing our Gaussians at the means outputted by K-means or K-medioids. In our algorithm, we will initialize them at promising peaks with high prominence (topographic prominence).

After obtaining a set of distributions that match the magnitude distribution, we can use something like regression to convert each Gaussian into modal data.

## Analysis and Discussion
While the Approximate Regression Method seems promising, the LMA library used made it difficult to use techniques such as barriers to try and prevent $r$ and $g$ from going outside of their expected range. This meant that for complex signals with many potential modes, LMA would often end up in local minima with $r$ and $g$ outside of their expected ranges. Because of this, there are no tested results associated with this method that are presented in this report. Given more time, a better optimization algorithm could potentially be used to make this method more viable. The idea of representing the shape of the magnitude of the frequency response with a simpler function that approximates the shape seems promising and is a concept that is shared across both methods explored in this project.

<!-- \begin{figure}
    \centering
    \includegraphics[width=\linewidth]{figs/gaussian_modes.png}
    \caption{The frequency response of a set of 100 randomly generated modes.}
    \label{fig:original_freq}
\end{figure}

\begin{figure}
    \centering
    \includegraphics[width=\linewidth]{figs/gaussian_estimated_modes.png}
    \caption{The fit achieved by the Approximate Expectation Maximization method on the frequency domain data presented in Figure \ref{fig:original_freq}.}
    \label{fig:gmm_estimated_freq}
\end{figure} -->

![image](/assets/figures/mode_reconstruction/gaussian_modes.png)

![image](/assets/figures/mode_reconstruction/gaussian_estimated_modes.png)

The Approximate Expectation Maximization algorithm appears to work quite well. An experiment was conducted with 100 randomly generated modes, the frequency response of which can be seen in the first above figure. After running EM, we get the estimated distribution shown in the second above figure which closely resembles the original (up to scaling since the output of EM is a proper distribution that sums to 1). As EM often converges faster than gradient descent, this algorithm is likely to use less iterations than other gradient descent methods.

One of the major drawbacks of the current EM algorithm is that Equation 2 tends to have a much larger magnitude further away from the center than Gaussian distributions. This means that for signals with very few modes, the fitted Gaussians tend to have a very large spread since they have to account for the points on the graph associated with a mode that are far away but still have significant magnitude. Given more time, there are a few potential techniques that could be used to mitigate this as well as improve the accuracy of the algorithm overall:

* We could come up with a new distribution that can be described similarly to Equation 3 which shares a similar shape to Equation 2 over the domain from 0 to $\pi$. This would mean we are fitting distributions that are more similarly shaped than Gaussians to the actual modes we are looking for and would prevent the issue that occurs with sparse modes. One potential way to do this could be creating a distribution that is a union of multiple Gaussians with parameters that are weighted proportionally. This could be simpler than coming up with a completely different distribution since we can use similar equations for calculating responsibility and updating the distributions.

* We could apply some kind of transform to the frequency response that makes the modes appear more Gaussian-like. It's a little less obvious how this could be done but it's still a potential approach.

