---
title: More surface growth
subtitle: Let's make some tendrils
date: 2017-07-30 00:00:00
description:
featured_image: /assets/notes/tendrils/nw_comp_tendrils.v001.jpg
---

This is another application of the FindShortestPathSOP (FSP) on interesting input geo.

The input geo in this case starts with a grid of quads, with each quad scaled down to their centroidscentroids via the ExtrudeSOP. Like this:

We want all the FSP start points at one end of the input grid, in order to direct the overall flow of the network. I used the following VEX function to do this:

relpointbbox(0, @P).x < 0.001

This selecting all the prims on one edge of the grid along the x axis. We then select other prims randomly with a GroupSOP. As a final step, we remove all the ends that are too close to the starts with a similar bounding box function:

relpointbbox(0, @P).x > .6;

We subdivide the curves to smooth them, and use the typical common combination of a FuseSOP followed by a PolyPathSOP to convert the overlapping paths into a single, tidy network.

We also want to create some accumulating attribs, as per this example, but with more than one start point in the network we must first restore the start ponit group, which was destroyed along the way. Like this:

Now, to animate. As in the previous example we carve by our accumulating attrib and, hey presto, animated growth. However, the result feels very linear, particualrly where the growth comes to an end at the terminal prims.

We can adjust the animation be stretching and compressing the arrival time attribs. The greater the distance the slower the animation (and vice versa), while non-linear distances will create non-linear animation (easing).

In order to arrive at our desired result we first need to isolate all the terimal prims, we can do that with our previous tip finding trickary and a group promote.

We then use this command to stretch the arrival time at the terminal prims, by adding to it using the curves intrinsic uvs. We also bias the stretching so that it begins aggresively before slowing down toward the end of the prim. This creates a pleasent easing effect:

To achieve the particular finish on the main tendrils I scatter points on the pre-carved curved network, and then sample the animated prims to see how close our scattered points are to the original prim.

To create the larger circles on the ends of the terminal prims, we need to know the ratio of prim length pre and post carve, which will grow to one as the length of a prim arrives at its pre-carve length.

We would also like to know the prims that are at the terminal level, which prims are at the ends of the network.