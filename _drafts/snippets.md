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

How to get contact between an animated card and the ground for an entire sequence.
isolate tires to prevent heavy geo - trail - timeshift - vdb/poly - group by object