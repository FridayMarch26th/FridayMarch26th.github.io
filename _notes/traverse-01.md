---
title: Traverse 01
subtitle: Let's make some tendrils
date: 2017-07-30 00:00:00
description:
featured_image: /assets/notes/tendrils/nw_comp_tendrils.v001.jpg
---

The effect is largely a product of using a FindShortestPathSOP (FSP) on interesting input geo.

In this instance I've used a TetConformSOP on a sphere, it provides an appropriate internal structure that requires minimal cleanup. The nature of the curves that are fed into the FSP plays a srong part in the look of the subsequant tendrils.

For the tendrils to grow upward, we sort the bounding box y, making point 0 the lowest point in the input geo. THis becomes our start group, we select random end groups and a noisy cost attrib to avoid the FindShortestPath operation to help introduce organic detail.

![alt text](assets/notes/tendrils/tendril_hard.jpg)


The polylines are then smoothed with a subdivisionSOP (or a resample SOP with "resample by polygon wdge" ticked).

![alt text](assets/notes/tendrils/tendril_smooth.jpg)

It's important to remember that, at this point, there are many overlapping prims, the differt output paths from the FSP op. We must therefore fuse the curves together, and then apply a polypath, to remove overlapping prims and create the necessary polypath network.

We now have a useful set of curves that we can skin.

In preparation we create a couple of useful attribs using the AttribFillSOP, values that accumulate over the network from points that we specify that we specify along the networks of curves (EdgeTransport is also useful for this).

We can use arrival time from the start point, and arrival time from the points at their tips.

We will use this accumulating attribute to dirve everything from tendril thickness to animation.

![alt text](assets/notes/tendrils/nw_comp_tendrils.v001.jpg)

We find 

FOr starts, we can remap teh time from leaf-level



