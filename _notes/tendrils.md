---
title: Tendrils
subtitle: Let's make some tendrils
date: 2017-07-30 00:00:00
description:
featured_image: /assets/notes/tendrils/nw_comp_tendrils.v001.jpg
---

A wide variety of branching/tendril effects can be made with the FindShortestPathSOP (FSP).

The process is generally as follows:

1. Create interesting input geometry.
2. Allocate points to start and end groups.
3. Cost (typically noise).
4. FSP.
5. Resampling.
6. Cleanup.


In this example we satisfy step one with a TetConformSOP applied to a sphere. This provides a pleasing and *somewhat* regular internal structure that requires minimal cleanup. The VoronoiFractureSOP also offers interesting results, or boolean ops, the list goes on.

For the tendrils to grow upward from an origin, we sort our sphere by the bounding box y, making point 0 the lowest point in our input geo. This single point becomes our start group, and we select random ends with the GroupSOP, with a noisy @cost attrib to encourage wayward organic detail. The result looks like this:

![alt text](assets/notes/tendrils/tendril_hard.jpg)

The polylines are then 'smoothed' with a SubdivisionSOP (or a Resample SOP with "resample by polygon edge" ticked). Retaining hard edges while increasing curve resolution can be accomplished with and EdgeDivideSOP, or by adjusting the parms on the SubdivisionSOP. We now have this:

![alt text](assets/notes/tendrils/tendril_smooth.jpg)

It's important to note that, at this point, we have many overlapping prims that need to be combined and reduced into a single curve network. WHile the resampling methods above are easily cleaned up, not all methods have the same advantage (despite offering interesting artistic outcomes). For us, FuseSOP the curves together will combine identically positioned points, then applying a PolyPathSOP will combine prims that share all points. 

Voila, an interersting network of curves. As a final measure we can use a MatchSizeSOP to position our point 0 at the world origin. Useful.

Next, skinning and animation.

In preparation for these steps we create a couple of useful attribs using the AttribFillSOP. These create values that accumulate over the curve network from specified points. TheEdgeTransportSOP can also be used to create these attributes, as can the output Cost Attrib from the FindShortestPathSOP itself. Each yields slightly different possibilities that might be worth a poke. 

Here we can see an arrival time attrib from a boundary group set to our start point, and another from the points at the network's terminal tipsnetwork. This second group we can derive by identifying all points with only one neighbour that are not part of the starts group:

!*starts
neighbourcount(0, @ptnum) == 1


![alt text](assets/notes/tendrils/nw_comp_tendrils.v001.jpg)

For animation I've used the "Clip trick" outlined on Matt Estela's CGWiki. This has now been replaced in Houdini 20.5, and a CarveSOP that can now carve by attrib. However, Matt's trick remains interesting as it allows us to inspect the values what we're using to carve, by viewing the geometry in "Carve space".

We can control the speed of growth by scaling arrival time attribute before the carve. There's more on that in this write up.

For skinning thickness we use arrival_time_from_tips. This ranges from 0 at the tips (no thickness at all), growing thicver as we move toward the root. With a typical combination of min/ max/ chramps etc, we can dial in the tips to taste.

Also worth note is the AttribFill method "Interpolate". this can smoothly blend between different values at boundary groups, offering a different way to mitigate the common problem of defining nicely tapered tendril tips with no weird discontinuities.

Done. Ish. The rendered example seen at the top of this age uses a SweepSOP. It's scaled by a @pscale that is derived from our adjusted arrival time, and nothing more. It's obviously going to skin prims independantly and create the possibility offor artifacting around interesecting prims or at junctions. This may or may not be an issue.

If a more cohesive network is required then there is always the VDBFromPolySOP, at a resolution eye-wateringly high enough to produce the fine detail of pointy tips. This is often slow to process, but I've sees this strategy used many times in production. Cache GEO out on the farm, and thank your lucky stars that you're not working with FLIP. :)


