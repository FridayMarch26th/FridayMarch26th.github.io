---
title: 'Code Snippets'
subtitle: 'Houdini odds and ends that I wanted to stick somewhere.'
date: 2018-07-30 00:00:00
description: Houdini odds and ends that I wanted to stick somewhere.
---

**Random axis vector**

vector2 r = rand(@ptnum);
v@dir[int(r.x * 3)] = r.y < 0.5 ? -1 : 1;


**Don't forget arbitrary group syntax in s a VEX function.**

int points[] = expandpointgroup(0, sprintf("* ^@startCode!=%s ^@destCode!=%s", s@destCode, s@startCode));

Plum the google notes

Clip by noise in H20.5

Igor's volume stuff.

Extract transform > transform adjacent pieces. Reduce the transform of a piece down to a point, make sure naming is same across the nodes that require naming otherwise it will result in weird deformations.
![Extract transform](/assets/misc/transform_pieces.png)

One to come back to. CP{S}:
https://www.youtube.com/watch?v=FyhR1VYJ3N4

This is the best beginners guide to Solaris I've seen:
https://www.youtube.com/watch?v=WfC16LYYIAw

I ran into this alongside the description "classic technique, nicely explained". It is totally that. https://youtu.be/1gIZJ7xmJ_g?si=9SPlttfx8F0npdUo

YouTubeYouTube | CG Forge
Fast Moving Pyro 

Creating stylized FX from Mikros:
https://www.youtube.com/watch?v=2ws-oTnUDcg

YouTubeYouTube | Houdini
Paris HUG Mikros Animation Presentation 


Cool oceans tricks and some dire audio quality... https://www.youtube.com/watch?v=TdxoTouaEFw&t=654s

YouTubeYouTube | Houdini
Evolving Oceans Toolset | Hristo Velev | Horizon HUG Sofia 

Not Houdini, but it's where my brain goes where it needs to put things. Interesting comparison of comp DoF solutions:
https://www.youtube.com/watch?v=HTM59OFuQfQ

YouTubeYouTube | Pixelfudger
Nuke - Realistic Depth of Field Defocus. Complete Guide for CLEAN edges! 

Detangle
https://www.youtube.com/watch?v=fI9UHomoeuk
