---
layout: post
title: New project - Vertextracer!
tags: [blog, projects]
gh-repo: shot511
gh-badge: [follow]
---

I would like to announce that on my GitHub account there is a new project called **Vertextracer**!  

This is a typical raytracer, for which I was preparing to write a longer time, but paid off, because it was a very enlightening experience - I have deepen my knowledge about Computer Graphics. In my opinion (after the experience with raytracer and OpenGL renderer) I consider that everyone who starts his adventure with Computer Graphics, should start from programming the raytracer.

!["Vertextracer - sample render"]({{ site.baseurl }}/img/vertextracer-sample-render.jpg){: .center-image}

Link to the project can be found in the [**Projects**]({{ site.baseurl }}/pages/projects) page. You can download the source, compile it yourself and try it out. As for the Vertextracer it has the following features:

*   Loading scenes from a file
*   Support for multiple sources of light (directional, point)
*   Rendering shadows
*   Support for various materials (opaque, transparent, refractive)
*   Load the textures and models
*   Texture filtering (by default the nearest filtering, bilinear filtering)
*   Rendering the colors of the sky (Atmospheric Scattering)
*   Anti-aliasing (adaptive, stochastic, FXAA)
*   Multithreading

I would also point out that this is not my main project and I will not be held unless I have to. However, I encourage you to report important bug fixes and new features. If it will be a lot of interest in the future, I will write a tutorial about how to create a raytracer from scratch.
