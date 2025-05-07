---
title: Tendrils #1
subtitle: With the FindShortestPathsSOP
date: 2024-07-30 00:00:00
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

In this example we satisfy step one with a TetConformSOP applied to a sphere. This provides a pleasing and somewhat uniform internal structure that requires minimal post-processing to prepare it for the FSP.

*The VoronoiFractureSOP can also offer interesting results, as can boolean ops, the RemeshSOP with adaptive edge lengths (perhaps followed by a DivideSOP to compute the dual graph for nice cellular structures), the list goes on... BUT, the TetConformSOP is hard to beat for pleasing results out of the box.*

For the tendrils to grow up and out from an origin, we sit our sphere on the ground with a MatchSizeSOP and then sort the points by proximity to the world origin. Point 0 is now the closest to the origin and becomes our start group, while we select random ends with the GroupSOP, with a noisy cost attrib to encourage wayward organic detail. The initial setup looks massively uninteresting...

<div class="gallery" data-columns="2">
	<img src="/assets/notes/tendrils/tendrils_initial_sphere.jpg">
	<img src="/assets/notes/tendrils/tendrils_initial_setup.jpg">	
</div>

...but an FindShortestPathSOP later (plug in start/ end groups, the cost attrib, and output paths "From any start to each end"), and the The result looks *marginally* more interesting:

![FSP](/assets/notes/tendrils/tendrils_hard.jpg)

The resulting polylines are then smoothed with a SubdivisionSOP (or a Resample SOP with "resample by polygon edge" ticked).

*Adjustment here can produce interesting variations, particularly where filtering causes terminal prims to fan out. We could also retain the angular paths while increasing curve resolution with an EdgeDivideSOP, or by adjusting the parms on the SubdivisionSOP... Notes for another time.*

We now have this:

![FSP with a bit of post-processing](/assets/notes/tendrils/tendrils_smooth.jpg)

Flowy. It's important to note that at the moment we have many overlapping prims that need to be combined and reduced into a single curve network. Thankfully, our methods of subdivision have retained the spacial similarity of output points (not all methods do this), which makes cleanup straightforward.

A FuseSOP will combine identically positioned points, while a PolyPathSOP will combine prims that share all points.

![Cleanup](/assets/notes/tendrils/tendrils_cleanup.jpg)

Next, skinning and animation.

For these steps we will create a couple of useful attribs with an AttribFillSOP (one now, one later). This SOP creates gradients that accumulate over the curve network from points specified in boundary groups.

*TheEdgeTransportSOP can also be used to create these attributes, as can the output cost attrib from the FindShortestPathSOP itself. Each yields slightly different possibilities that are worth a poke.*

![Cleanup](/assets/notes/tendrils/tendrils_arrival_time_from_roots.jpg)

Above we can see an arrival time attrib from a boundary group that includes our start point, (in this case an easily specified root at point 0), which is used for animated growth out from the root. It is predominantly red because the value of the attrib extends well beyond one.

For animation I've used a "Carve by attribute" trick outlined on Matt Estela's CGWiki (it's mentioned [here](https://tokeru.com/cgwiki/HoudiniFAQ.html#how_do_i_carve_lots_of_curves_at_different_rates); although I swear there was a more complete explanation of it at one point). This has now been replaced in Houdini 20.5, with a CarveSOP that IIRC features native support for carving by attrib. But we're still on H20 at work (Boooo). That said, a useful side effect of Matt's trick is that it allows us to inspect the values what we're using to carve, by viewing the geometry in "Carve space" before the original point positions are restored.

<div class="gallery" data-columns="2">
	<img src="/assets/notes/tendrils/tendrils_carve_setup.jpg">
	<img src="/assets/notes/tendrils/tendrils_anim.gif">	
</div>

*We can also control the speed of growth by scaling our arrival time attribute before the carve. There's more on that in another write up.*

For skinning thickness we use a second AttribFillSOP, this time accumulating from the tips after the carving animation has trimmed away as yet un-grown geometry. This group is created by identifying all points with only one connected neighbour:

```neighbourcount(0, @ptnum) == 1```

We now have a value that ranges from 0 at the tips (no thickness at all), growing larger as we move along the curves toward the root. With a typical combination of min/ max/ chramps etc, we can dial in tips to taste.

<div class="gallery" data-columns="2">
	<img src="/assets/notes/tendrils/tendrils_arrival_time_from_tips.jpg">	
	<img src="/assets/notes/tendrils/tendrils_thick.jpg">	
</div>

*Also worth note is the AttribFillSOP method "Interpolate". This can smoothly blend between different input values present on boundary group attribs, rather than a value that increases indefinitely from the boundary. This offers a different way to mitigate the common problem of defining nicely tapered tendril tips.*

And we're done.

Ish.

The example seen at the top of this page is rendered with Arnold's interpreting a @pscale attrib applied to a set of curves, which is remapped from our adjusted @time_from_tips attrib. The operation is obviously going to skin prims independantly and create the possibility of artifacting around interesecting prims or at junctions. Sometimes fine, other times not.

If a continuous skin is required then there is always the VDBFromPolySOP, at a resolution eye-wateringly high enough to produce the fine detail of pointy tips. This is often slow to process, but I've seen this strategy used many times in production. Cache GEO out on the farm, render with something that can swallow the resulting geometry, and thank your lucky stars that you're not working with FLIP. :)