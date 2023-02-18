---
layout: post
title:  "Start of 420 Capstone Project"
date:   2023-01-08 00:06:11 -0800
categories: 420-Capstone
---

## Original Proposal：

Traditional resonator effects work by passing a signal through a feedback system that delays the signal by an amount inversely proportional to the desired fundamental frequency. This causes frequencies in the signal that align with the delay time to resonate as a result of constructive interference. This results in the harmonics of the fundamental frequency being amplified while out of phase frequencies are attenuated. Another variant of the resonator does something similar except instead of adding the signal to itself, it subtracts the signal from itself. This causes the odd harmonics of the frequency at half of the fundamental frequency derived from the delay amount to resonate. These are both powerful tools that allow us to create tonality from otherwise atonal sounds as well as perform basic physical modeling of real-life instruments (e.g., the Karplus-Strong algorithm). They also benefit from being very simple to implement and are both very computationally efficient. However, these methods are quite limited in that they only allow us to generate resonances at harmonics of a particular frequency and we have neither control over the number of harmonics that resonate nor control over the amplitudes of each resonant frequency. What if we want to create a resonator that resonates like an actual musical instrument or want to define our own made-up resonant space? This idea is not necessarily new. People have been using physical modeling to synthesize various real-world instruments and resonant spaces for a while. Modal synthesis uses resonant filters to simulate vibrating objects and can be used to get very organic sounding effects. However, to my knowledge, there is no general-purpose resonator that can be used to easily build resonances based off of actual sounds—let alone create an instrument using the modal synthesis approach.


The goal of this project is to create a library in the Rust Programming Language that will allow us to define a resonant space by passing a sample that will determine the frequency response of the system. The sample could be anything from an instrument sample, such as a flute, or a reverberant impulse response. In addition, we will have the ability to edit the space by adding and removing individual resonators, changing their amplitude, and shifting their pitch. To create the resonance, we will use a dynamic number of resonant band-pass filters (bounds will be something like between 0 and 2000). The band-pass filters will be minimum phase IIR filters which will enable us to efficiently do the filtering operation as well as be a more accurate model for real-world resonances. The main advantage of using this method over a conventional convolution model is that we will have complete control over all of the resonances in the space including amplitude and frequency and we can update any of these values in real-time without significant overhead. This is important if our use-case is to be able to play notes by passing MIDI data. Another, more creative, use case is that we can theoretically do complex morphing between different resonant spaces which may be useful for something like simulating the change in timbre of an instrument at different registers. The main disadvantage of using this method over convolution is that it may be less performant and we won’t get the same level of detail as we don’t have a way to control the phase of the resonances. Either way, the goal of this library isn’t to be a better version of convolution, it’s to be an interesting and less restrictive way of doing modal synthesis.


The main reason for writing a library instead of creating a VST or a max object is that in the future, maybe the library can be used to construct both. In addition, I will be quite busy this semester and writing a VST GUI in Rust is something I want to properly implement a library for in the future. This means that in the current state, creating the GUI for a complicated project like this would take the majority of the time and would not be done properly anyways. For a demonstration, I will likely create a live demo with real-time audio and a simple CLI. I’ll likely include some basic visuals to show what is happening, but they won’t necessarily be interactive.


## Initial Approach:

## Todo list:

- [ ] Drawing bode plots

- [ ] Implementation of the band-pass filters

- [ ] Converting an arbitrary sample into a resonant space

- [ ] Creating a demonstration of the result


To start out, in order to debug things easily, I want the ability to display bode plots for a given transfer function in the Z-domain. This will be a useful tool to have for the next sections.


As mentioned in the proposal section, the resonance will be created using a dynamic number of minimum-phase IIR band-pass filters. These will be implemented by using an inverse Z-transform on the generalized conjugate pole pair to generate the generalized difference equation for a simple band-pass filter. This will result in a minimum phase band-pass filter that mimics real-world resonant systems (like a mass-spring system). The main problem to solve here is finding an optimal strategy for performing the calculation. We want to maximize performance while still ensuring the quality of the outputted sound (mainly have to worry about floating point inaccuracy since, to get high resonance, we have to push the poles close to the unit circle in the complex plane which may result in system instability). The filters could be made more versatile by adding the option to include an extra pair of conjugate zeros behind the poles to widen/narrow the range of affected frequencies. This would add more complexity but could potentially but used to get better fits for given samples.


One of the major challenges is going to be converting a sample into an array of band-pass filters. Most likely, this will work by passing a sample and a frequency range, finding the peaks in the given frequency range, and fitting a filter to the peak. There are two methods I can think of for doing this somewhat efficiently off the top of my head. We could use a stochastic gradient descent algorithm that starts with an initial guess and iterates a few times towards a better solution. Another method could be some sort of least-squares linear algebra technique in either the frequency or time domain which could be more efficient than the previous idea. Either way, this particular operation does not have to run particularly fast since it is not a real-time operation.


As mentioned in the proposal section, the demonstration will be something with real-time audio processing using a basic CLI. Likely, I’ll want to be able to play notes for some instrument, create some sort of resonant space, and probably show some interesting creative use-cases. If there is enough time, I’ll hopefully be able to implement a simple GUI for the demonstration binary. Either way, the final contents of the demonstration will be determined when the previous steps are completed.


## Suggested Outcomes:

My goal with this project is to develop a library that I will be able to use to construct a VST effect as well as use in-general to do some interesting sound synthesis. In addition, there may be interest in developing a max object by hooking into the library from C++. Audio programming in Rust is not well developed even though it meets the same performance requirements for real-time audio processing as C and C++ and has the added benefit of completely preventing an entire class of bugs that are typically associated with low level languages. This will be my first time programming a real-time audio system so it will be a pioneering project to help me figure things out. In the future, I would like to continue building interesting audio software on top of Rust.

