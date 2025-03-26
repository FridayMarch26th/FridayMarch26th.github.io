---
title: Velocity Volumes
subtitle: POPs along a path
date: 2017-07-30 00:00:00
description:
featured_image: /assets/notes/velocity/velocity_poster.jpg
---

![The result](/assets/notes/velocity/velocity_poster.gif)

Some solutions to a recent request for sending things along curves.

The input geo we start with is here is the same as in [this example](/notes/tendrils). Cleaned up FSP generated curves provide a nice foundation for this effect.

Now we need POPs, spwaned from animating input geometry. Same as above, curves carved by an arrival time gradient starting from a single root point.

The actual guts of the motion is very straightforward, a velocity volume around the entire curve network that advects the POPs. We use volume wrangles to adjust the velocity, with the curve network fed into the wrangles' second input. The network for what might look a bit like this:

The network for that might look a bit like this:

![The result](/assets/notes/velocity/velocity_network.jpg)

As for what's in the wrangles, we combine two vectors:

The first vector pushes the particle toward the closest point on the curve network:
```
vector pos = minpos(1, @P);
@v += normalize(pos - @P) * chf("Magnitude");
```

The second vector pushes the particle along the curve. 
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

You could even separate the effect into multiple volumes if you wanted a clearer separation of the various vectors, which would also give you access to adjusting individual force components from within POPs, but in the end I combined them inside a single volume prior to simulation for the sake of not looking up the velocity info' more times than was necessary.

An aside... In early tests we quickly fell into the problem that is this: When pushing particles along a network of curves, the obvious place to spawn them from is at the root. However, the annoyance of this is that with each successive generation of branches the distribution of particles grows thinner and thinner. Easy to spot, potentially difficult to mitigate nicely.

So here, by generating points over the entire curve network, rather than only at the origin, we can keep a nice volume of particles moving along.