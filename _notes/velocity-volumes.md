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

Our final example is more involved than the other two. It used a tetconform Based curve similr to the one found here.

This time, rather than sending the energy across the cruve network using attributes, we're going to use explicit particle positions sampled form the incoming network with some very common and useful functons.

The crux of the effect is to send a point alng a prim with primuv. In this example we add to the prim uv over time to push the particle along. Easy peasy.

But what happens when we get to the end of the prim? In the previous example we repeatedly cycle over the same prim, buty we need to make descisions about where to go next in order to move the point along the entire network.

We must first source some points at the origin, our point zero, and lt them decide which of the prims connecting to that ponit that travel along. We can find a list of connected prims with the function primpoints(). We pick one prim from this list,and then primuv our way toward the end.

But what about when we approach a function, more descisions required. Any point with a uv value above one has effectively moved beyond the end of the prim it's been trvelling along. Now, in much the same way as the initial descision, we can now query the prims connected to this final point and start the process again. We pick a new prim, we reset the uv, we continue onward.

The only additional constraint is waht to do if the point is a terminal, if it has now connecting prims. Well... In which case there's nowhere to go and the prim can be safely killed off.

NOW. One more thing. As mentiuoned earlier a common problem with this effect is that the density of particles reduces as the journey onward, the particles thin out through suvsequant generations of branches, until there aren't many left toward the leaves.

So rather than having a particle decide which branch to travel down, why not have it spawn subseqaunt particles and effectviely travel along all child curves at the same time! It might come in handyt for something...

We therefore create enough new points to satisfy the numeb of connected outgoing prims, and then cycle though those prims to send the new particles on their way.