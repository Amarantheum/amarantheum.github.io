---
layout: post
title:  "Scaled Resonator Builder"
date:   2023-03-23 00:14:11 -0800
categories: modal-synthesis
---

## Summary

There was some difficulty getting the complicated regression model to work consistently so in order to ensure a working demo for the presentation, it was decided to keep things simpler. Library documentation can be found [here](https://amarantheum.github.io/GPResonatorLibrary){:target="_blank"}.

### Goals Completed

- [x] Obtain a working sample-to-resonator-array implementation

## Scaled Resonator Builder

In order to simplify the process of obtaining a resonant space from a sample, instead of trying to do non-linear regression, a simpler method was proposed. This method sets the decay of each resonator to the same decay amount and scales the gain of each resonator by the height of the peak in the frequency spectrum corresponding to the resonator. This was able to recreate resonant spaces from sounds fairly well. In the future, it would still be good to explore regression techniques since they still recreate spaces more accurately than the simpler scaled method. Going forward, a demo project will be made that makes use of the resonator library to do some real-time audio processing.

[previous-post]: https://amarantheum.github.io/modal-synthesis/2023/02/17/non-linear-regression.html