---
title: Traversal Attribs
subtitle: Let's make some tendrils
date: 2017-07-30 00:00:00
description:
featured_image: /assets/notes/traversal-pops/nw_comp_traverse_pops.v001.jpg
---

![The finished result](/assets/notes/traversal-pops/traversal_pops-poster.gif)

The starting curve network used in this effect is outlined [here](/notes/tendrils). It gives us a single root from which the entire network emerges:

![Root](/assets/notes/traversal-pops/root.jpg)

In [this example](/notes/traversal-attrib) we sent energy along the curve network using attribs, this time we're going to use POPs.

The crux of the effect is to send a point along a prim with a common VEX function, primuv(). Each polyline prim has an intrinsic value along it ranging between zero and one. We can use this value to sample the position of any point along that prim, and we can move points along prims by offsetting that value.

![PrimUV](../assets/notes/traversal-pops/primuv.gif)

Matt Estela has a longer write up of the extremely useful primuv(), [here](https://tokeru.com/cgwiki/JoyOfVex19.html).

So with this, we can scatter a point onto a prim, and then send it along that prim by adjusting a "U" attrib. It's not unlike what a combo of the ScatterSOP (with @sourceprim and @sourcepimuv attribs) and an AttributeInterpolateSOP might do, but what happens when we get to the end of the prim? We need to pick the next one. Here we go.

This is POPs network. We have a PopLocation, a clump of nodes that define the traversal logic, and a couple of nodes that undertake replication (currently bypassed, we'll get there in a bit).

![Overview](/assets/notes/traversal-pops/traverse_pops_overview.jpg)

We must first source some points at the origin, our point zero, and lt them decide which of the prims connecting to that ponit that travel along. We can find a list of connected prims with the function primpoints(). We pick one prim from this list,and then primuv our way toward the end.

But what about when we approach a function, more descisions required. Any point with a uv value above one has effectively moved beyond the end of the prim it's been trvelling along. Now, in much the same way as the initial descision, we can now query the prims connected to this final point and start the process again. We pick a new prim, we reset the uv, we continue onward.

The only additional constraint is waht to do if the point is a terminal, if it has now connecting prims. Well... In which case there's nowhere to go and the prim can be safely killed off.

NOW. One more thing. As mentiuoned earlier a common problem with this effect is that the density of particles reduces as the journey onward, the particles thin out through suvsequant generations of branches, until there aren't many left toward the leaves.

So rather than having a particle decide which branch to travel down, why not have it spawn subseqaunt particles and effectviely travel along all child curves at the same time! It might come in handyt for something...

We therefore create enough new points to satisfy the numeb of connected outgoing prims, and then cycle though those prims to send the new particles on their way.