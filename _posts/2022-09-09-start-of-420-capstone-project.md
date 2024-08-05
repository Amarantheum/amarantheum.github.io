---
layout: post
title:  "GP Resonator Library Concept"
date:   2022-09-09 00:08:51 -0800
categories: 420-Capstone
---

## Project Idea

The goal of this project is to create a resonator library where the user can re-create any resonant space ny building a filter composed of a large number of resonating filters. Ideally, this filter should be able to process incoming audio in real-time as well as be modifiable in real-time. This library can be used as a backend for VSTs or MAX objects. To process audio in real-time, simple IIR band-pass filters will be used to create the resonances. To facilitate the creation of resonant spaces, extra tools will be added to the library that generate resonant spaces from samples.

![image](/assets/images/GPResonator.drawio.svg)