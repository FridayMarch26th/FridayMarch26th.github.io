---
title: Constraints
subtitle: I totally did not know that a SopSolver with multiple outputs was right there.
date: 2018-09-27 00:00:00
description: Handy Shortcut
featured_image: /assets/notes/feature_constraints.jpg
---

![Screengrab](/assets/tech/constraints.gif)


Working in DOPs often involves modifying constraint geometry in a SOP Solver, the setup of which can be a *touch* on the tedious side. I stumbled across a tab menu entry that takes care of that. Shortly after I stuck the news of it on Twitter, and was shown a nicer solution...

**ᴍᴀᴛᴛɪᴀꜱ ᴍᴀʟᴍᴇʀ**
*@3Dmattias*
May 21, 2023
A way around that is to use two sopsolvers on running on Geo the other on Con. In the first one you load in the Geo AND the Con. Do Process and output the Geo as usual. But also make a null for the Con. Then in the next sopsolver you object merge the null and just send it out.

**Luke Van 🥴**
*@wobblypictures*
May 21, 2023
That’s what I’ve been having to do yeah. It’s a cool preset but given the first thing you’ll try do do is rebuild the sop constraints immediately stopped in your tracks, it’s a little unintuitive