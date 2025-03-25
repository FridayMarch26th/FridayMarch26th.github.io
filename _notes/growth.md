---
title: Surface growth
subtitle: Another day, another tendril setup.
date: 2017-07-30 00:00:00
description:
featured_image: /assets/notes/growth/nw_comp_growth.v001.jpg
---
![The finished result](/assets/notes/growth/growth_preview.gif)

The input geo in this case starts with a grid of squares, with each square then scaled down to their centroids via the ExtrudeSOP, followed by a FuseSOP to remove overlapping points. Essentially this, at a suitably high resolution:

![Growth base](/assets/notes/growth/growth_base.gif)

We want all the FSP start points from along one open edge of the input grid. This VEX will do that:

```@group_starts = relpointbbox(0, @P).y == 0.0```

We then select end points randomly with a GroupSOP. As a final step, from the end group we remove points that are too close to the starting line with a similar bounding box function:

```@group_ends *= relpointbbox(0, @P).y > 0.6```

Next, in the same manner as [this example](/notes/tendrils), we subdivide the curves and use the typical combination of a FuseSOP followed by a PolyPathSOP to convert the overlapping paths into a single, tidy curve network.

![Subdivided Ggrowth](/assets/notes/growth/growth_subd.gif)

Similarly, we also need our accumulating attribs, but with more than point 0 point in the network it's worth restoring the original group procedurally, which was destroyed along the way. So what we now have looks a little like this:

![The story so far](/assets/notes/growth/growth_sofar.gif)

Now, to animate. As in the previous example we carve by our accumulating attrib and, hey presto, animated growth. However, the result feels very linear, which is jarring where the animation comes to an abrupt stop at the terminal prims.

![Linear Anim](/assets/notes/growth/growth_preslow.gif)

We can adjust the animation by stretching and compressing the arrival time attribs. Larger changes in arrival time along a prim will slow animation (and vice versa), while non-linear changes will create non-linear animation (easing).

SO. We first need to isolate the terminal prims. We can find them by identifying terminal points with this:

```@group_tips = neighbourcount(0, @ptnum) == 1 && !inpointgroup(0, "starts", @ptnum)```

And then we feed the resulting point group into a GroupPrommoteSOP to prims. Then it's a straight forward case of adding to the original arrival time over the length of the prim, with a chramp parm to introduce bias.

![Linear Anim](/assets/notes/growth/growth_biasattrib.jpg)

The result of that adjustment is some nice easing as the wandering tendrils arrive at their destinations.

![Linear Anim](/assets/notes/growth/growth_postslow.gif)

What remains is to scatter points on the pre-carved curved network, and then either AttribInterpolateSOP the animated values to those points to drive colour and scale, or perhaps measure the distance between the scattered points and the carved curve network - the greater the distance from the curve network, the less active the ponits will be.

For the larger circles on the ends of the terminal prims, we need to know the ratio of prim length pre and post carve, a value that will increase from 0.0 to 1.0 as the length of a prim arrives at its pre-carve state.