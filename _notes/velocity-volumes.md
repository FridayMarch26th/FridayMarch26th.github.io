---
title: Velocity Volumes
subtitle: Pushing POPs along a path
date: 2024-07-30 00:00:00
description:
featured_image: /assets/notes/velocity/velocity_poster.jpg
---

![The result](/assets/notes/velocity/velocity_poster.gif)

A solution to a recent request for sending things along curves.

The input geo we start with is here is the same as in [this example](/notes/tendrils). Cleaned up FSP generated curves provide a nice foundation for this effect.

Now we need POPs, spawned from the animating-in input geometry. Again, same as [this example](/notes/tendrils), curves carved by an arrival time gradient starting from a single root point.

The actual guts of the motion is very straightforward, a velocity volume around the entire curve network that advects the POPs. We use volume wrangles to adjust the velocity volume, with the curve network fed into the wrangles' second input so that we can sample it.

The network for that might look a bit like this:

![The result](/assets/notes/velocity/velocity_network.jpg)

As for what's in the wrangles, we're combine two vectors:

The first vector pushes the particle toward the closest point on the curve network:
```
vector pos = minpos(1, @P);
@v += normalize(pos - @P) * chf("Magnitude");
```

The second vector pushes the particle along the curve (you'll need an OrientAlongCurveSOP to create the necessary normal attrib). 
```
int pt = nearpoint(1, @P);
v@v += point(1, "N", pt) * chf("Magnitude");
```

You could also remove all velocity inside the curve network by sampling a surface volume with a bit of this:
```
float vs = volumesample(1, "surface", @P);
@v *= vs > 0;
```

Notice that we're adding forces with the first two operations, then we're multiplying them down to nothing with the third.

You could split the effect into multiple volumes if you wanted a clearer separation of the various vectors. This would give you access to adjusting individual force components from within POPs, for example, but in the end I combined them inside a single volume prior to simulation for the sake of not looking up the velocity info' more times than was necessary.

An aside... In early tests we quickly fell into the problem that is this: When pushing particles along a network of curves, the obvious place to spawn them from is at the root. However, the annoyance of this is that with each successive generation of branches the distribution of particles grows thinner and thinner. Easy to spot, potentially difficult to mitigate nicely.

So here, by generating points over the entire curve network as it animates in, rather than only at the origin, we achieve a nice volume of particles moving along along with the "animate in" effect.