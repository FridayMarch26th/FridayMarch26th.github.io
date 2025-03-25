---
title: Traversal Attribs
subtitle: Let's make some tendrils
date: 2017-07-30 00:00:00
description:
featured_image: /assets/notes/traversal-pops/nw_comp_traverse_pops.v001.jpg
---

![The finished result](/assets/notes/traversal-attrib/travesal_poster.gif)

This example starts with a 2D curve network, with the input to the FindShortestPathSOP created via a circleSOP and a VoronoiFractureSOP, with a bias favouring end points further away from the origin.

We then create a group of points that will be at the cutting edge of the energy as it travels along the network, and create an attrib to represent that energy

Next, an important step. We look up the nbeighbouring points, storing those downstream into an array attrib. This is completed before any animation begins, creating a lookup that we can refer to later we can avoid unnecessary computation. This creation of a lookup is interesting, and can use different metricsm ethods (coneangle etc). 

To create a regul pulse, we stike a beat every second

we then use the following code to pick the first child point in any points children array, and send it the energy we store. We then shuffle that child to the back of the array. We could also pick the child by random id, or indeed by whatever metric provides an interesting result.

Having selected tnew leadersa nd passe on the energy, we can now decay the energy across the network in order to fade the energy down to zero.