---
title: Tendrils #1
subtitle: Some common techniques for making wandering paths.
date: 2017-07-30 00:00:00
description:
featured_image: /assets/notes/tendrils/nw_comp_tendrils.v001.jpg
---

A wide variety of branching/tendril effects can be made with the FindShortestPathSOP (FSP). Here's one...

![The finished result](/assets/notes/tendrils/nw_comp_tendrils.v001.jpg)

**The general process is as follows:**

1. Create interesting input geometry.
2. Allocate points to start and end groups.
3. Cost (typically noise).
4. FSP.
5. Resampling and cleanup.
7. Animation, skinning, and whatever else you need tendrils for...

In this example we satisfy step one with a TetConformSOP applied to a sphere. This provides a pleasing and somewhat uniform internal structure that requires minimal fiddling with.

*The VoronoiFractureSOP can also offer interesting results, as can boolean ops, the RemeshSOP with adaptive edge lengths perhaps fed into a DivideSOP to compute the dual grpah for nice celular structures, the list goes on... BUT, the TetConformSOP is hard to beat for achieving good results out of the box.*

For the tendrils to grow up and out from an origin, we sit our sphere on the ground with a MatchSizeSOP and then sort the points by proximity to the world origin. Point 0 is now the closest to the origin and becomes our start group, while we select random ends with the GroupSOP, with a noisy @cost to encourage wayward organic detail. The initial setup looks massively uninteresting...

<div class="gallery" data-columns="2">
	<img src="/assets/notes/tendrils/tendrils_initial_sphere.jpg">
	<img src="/assets/notes/tendrils/tendrils_initial_setup.jpg">	
</div>

...but an FSPSOP later (plug in start and end groups, the cost attrib, and output paths "From any start to each end"), and the The result looks *marginally* more interesting:

![FSP](/assets/notes/tendrils/tendril_hard.jpg)

The polylines are then smoothed with a SubdivisionSOP (or a Resample SOP with "resample by polygon edge" ticked).

*Adjustment here can produce interestingvariations, particularly where filtering causes terminal prims to spread for extra frilly detail. We could also retain the angular paths while increasing curve resolution with ad EdgeDivideSOP, or by adjusting the parms on the SubdivisionSOP... Excercises for another time.*

We now have this:

![FSP with a bit of post-processing](/assets/notes/tendrils/tendril_smooth.jpg)

Nice and tendril-y. It's important to note that, at the moment, we have many overlapping prims that need to be combined and reduced into a single curve network. Thankfully, our methods of subdivision have retained the spacial similarity of output points, which makes cleanup straightforward.

A FuseSOP will combine identically positioned points, while a PolyPathSOP will combine prims that share all points.

![Cleanup](/assets/notes/tendrils/tendril_cleanup.jpg)

Next, skinning and animation.

In preparation for these steps we create a couple of useful attribs with an AttribFillSOP. This SOP creates values that accumulate over the curve network from specified points.

*TheEdgeTransportSOP can also be used to create these attributes, as can the output cost attrib from the FindShortestPathSOP itself. Each yields slightly different possibilities that are worth a poke.*

<div class="gallery" data-columns="2">
	<img src="/assets/notes/tendrils/tendrils_arrival_time_from_roots.jpg">
	<img src="/assets/notes/tendrils/tendrils_arrival_time_from_tips.jpg">	
</div>

Above we can see an arrival time attrib from a boundary group set to our start point, (which in this case is an easily retrieved point 0), and another arrival time attrib from the points at the network's terminal tips. This second group we can derive by identifying all points with only one connected neighbour:

```neighbourcount(0, @ptnum) == 1```

For animation I've used a "Carve by attribute" trick outlined on Matt Estela's CGWiki (it's mentioned [here](https://tokeru.com/cgwiki/HoudiniFAQ.html#how_do_i_carve_lots_of_curves_at_different_rates; although I swear there was a more complete explanation of it at one point). This has now been replaced in Houdini 20.5, with a CarveSOP with native supprot for carving by attrib. But we're still on H20 at work (Boooo), while a useful side effect of Matt's trick is that allows us to inspect the values what we're using to carve, by viewing the geometry in "Carve space" before the original point positions are restored.

<div class="gallery" data-columns="2">
	<img src="/assets/notes/tendrils/tendril_carve_setup.jpg">
	<img src="/assets/notes/tendrils/tendrils_anim.gif">	
</div>

*We can control the speed of growth by scaling arrival time attribute before the carve. There's more on that in this write up.*

For skinning thickness we use our @time_from_tips. This ranges from 0 at the tips (no thickness at all), growing thicker as we move along the curves toward the root. With a typical combination of min/ max/ chramps etc, we can dial in tips to taste.

Also worth note is the AttribFillSOP method "Interpolate". This can smoothly blend between different values at boundary groups (not just increasing from the boundary). This offers a different way to mitigate the common problem of defining nicely tapered tendril tips with no weird discontinuities.

And we're done. Ish.

The rendered example seen at the top of this page uses a SweepSOP. The tendril thickness scaled by a @pscale that is remapped from our adjusted @time_from_time, and nothing more. The operation is obviously going to skin prims independantly and create the possibility of artifacting around interesecting prims or at junctions. This may or may not be an issue.

If a continuous skin is required then there is always the VDBFromPolySOP, at a resolution eye-wateringly high enough to produce the fine detail of pointy tips. This is often slow to process, but I've seen this strategy used many times in production. Cache GEO out on the farm, render with something that can cope with the resulting geometry, and thank your lucky stars that you're not working with FLIP. :)