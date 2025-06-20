---
title: "Real-time Rendering of Water Reflections in VR"
date: 2025-06-19T21:29:01+08:00
# lastmod: 2020-03-06T21:29:01+08:00
draft: false
author: "Cristian"

resources:
- name: featured-image
  src: images/thumbnail-master-thesis.png

tags: ["virtual-reality", "rendering"]
categories: ["research"]

rssFullText: true

toc:
  enable: false
  auto: false

share:
  enable: true
---

Rendering accurate water reflections is crucial for achieving realism in computer graphics. Their integration with VR and AR technologies can further elevate experiences in sectors like education, engineering, and medicine.

<!--more-->

### Abstract ([Paper](https://repository.tudelft.nl/record/uuid:6a14ca81-a277-4a1b-b45f-b6ac50b45045))

Even though various monoscopic reflection techniques exist they either demand substantial computational resources or cannot be straightforwardly adapted to stereoscopic media, leaving a big hole in research concerning stereo-aware reflections and their applications. This thesis researches and analyses the different techniques for water reflections. Further, it makes significant strides in the development of stereo-consistent real-time water reflections for XR platforms. This is achieved by combining implicit functions and bounding volume hierarchies (BVH) into a unified system called Adaptive Hierarchical Signed Distance Fields Reflections (AH-SDFR). It was found that AH-SDF outperforms leading methods such as Hierarchical Screen Space Reflections (HSSR) and Reflection Probes in both visual quality and performance, and as a result, it offers a promising solution for XR applications. However, challenges persist, mainly due to the large memory demands of intricate 3D scenes and perceptual errors from inaccuracies in volume textures.