
---
title: Misc
subtitle: Stufgf I've encountered that I wanted to keep
date: 2023-02-10 00:00:00
---

![Reference pic](assets/genus.webp)

Find the "genus"(boundary groups) of geo:

string grp[] = detailintrinsic(0,"pointgroups");
i@total = len(grp)-1;

Flip bubbles
https://youtu.be/4JDNQtIYKjM?si=dJigh8pocSLCq28r




Diffuse white is 1.0 - and we have 2^4 stops abvoe this as head room for specular highlights. That is space to store things above values that appear as white in a photographic scene..

Quote from Gemii on ACES colour and speculr highlights:
It needs that approximately 16.0 value to push it through the tone-mapping curve and out to the monitor's physical limit.

We're not storing display referred colour, so colour started in a white that has a hard limit where our output values are saturated".
When we store in scene referred colour, wee are storing the values as they wuld be presented in the real world.

Nice display referred quote
"There’s no extra tone mapping or exposure logic needed — it’s “ready for screen.”"
