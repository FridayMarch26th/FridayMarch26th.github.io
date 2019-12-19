---
title: 'Paul Clements inspired growth doodad'
subtitle: 'Procedural blobz'
date: 2018-09-27 00:00:00
description: A piece of work from a friend that I wanted to reproduce.
featured_image: '/images/project-clements/clements (1).jpg'
---

![](/images/project-clements/clements (1).jpg)

I saw a spot from a top 3d artist [Paul Clements](https://twitter.com/paulclementstv). Growing blobs based on a voronoi diagram. Paul's a Cinema maestro, but I like Houdini...

The setup extracted all of the profiles from a fractured grid, and matched them with a curve. Various attribs bend and grow the curve, and then the profile is swept back along the curve to create the finished nodule.

For extra performance I had to avoid the non-compilable sweep sop, instead making my own from lower-level operators.

The original doodad is [here](https://t.co/3F441GJj3B).