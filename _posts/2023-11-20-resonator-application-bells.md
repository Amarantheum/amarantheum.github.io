---
layout: post
title:  "Modal Synthesis Library Application: MaxMSP and Bells"
date:   2023-11-20 00:00:00 -0800
categories: modal-synthesis
---

This is the first piece I've worked on that utilizes the modal synthesis library I wrote in Rust. Because MaxMSP was used to track the dancer and control the audio, a Max external was written in Rust to utilize the modal synthesis library.

The piece uses linear interpolations between different resonant spaces which are controlled by the dancer's position as well as distance between hands.

<iframe width="560" height="315" src="https://www.youtube.com/embed/ug2vfM7eUQk?si=3kmGtMbP_yGhi_Om" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>