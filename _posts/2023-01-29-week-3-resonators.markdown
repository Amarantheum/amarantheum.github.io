---
layout: post
title:  "Week 3 - Resonators"
date:   2023-01-29 00:21:32 -0800
categories: 420 Capstone
---

# Week 3 - Resonators

This week, I wrote an implementation for displaying Bode plots. The library works by taking a list of transfer function objects and generating a graph of the magnitude of the frequency response. The library then opens a window displaying the plot of the total frequency response of the transfer functions.

![image](./images/bode_plot_0.png)

In addition, I started implementing the resonators themselves by creating a ConjPoleResonator struct that represents a set of conjugate poles in the Z-plane. The documentation for the library can be found [here][gp-resonator-docs].

[gp-resonator-docs]: https://amarantheum.github.io/GPResonatorLibrary 