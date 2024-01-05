---
title: World Maps
subtitle: Rough production notes
---

## Intro##

A while ago I began to research a project based around high-resolution world maps. Dramatic camera work was planned, with the camera plummeting from outerspace to a few kilometers away from the earth's surface.

I was tasked with looking at solutions for 3D topographic detail, and planetary clouds.

Unfortauntely it's a project that never came to fruition, so we never reached the final frame, but enough work was completed to warrent some notes. It's also rare that encounter a contribution that isn't tedious social media banter, and with that perhaps bits of the following will be useful to a future someone.

Also, it's worth mentioning that some of the notes are specific to Arnold, being the render that would be used in prodction, but I've included them here because similar hurdles might need to be overcome regardless of renderer.


Topographic Detail - Digital Elevation Model - DEM

A helpful colleague pointed me toward DEM data sets that are available from the NASA website. They are essentially series of image tiles sourced from satelites scanning the globe at regular intervals. These can be used to drive displacement. One particular set offered a level of quality that satisfied the production resolution given the proximity of the camera to the earth's surface:

https://search.earthdata.nasa.gov/search/granules?p=C1711961296-LPCLOUD&pg[0][v]=f&pg[0][gsk]=-start_date&fi=ASTER&tl=1703889672.142!3!!

What a resource! But also, we've just downloaded shy of 23k images and around 400GB of data. Yowza.

REMAPPING
A lot of the work requires mapping one set of values (the image filenames) to another (a grid of primitives). This is broadly trivial, but any remapping requires a unique set of tricks to transform the input values into meaningful output.

Latitude and longitude are a set of coordinates that begin at the intersection between the prime meridian and the equator, extending vertically between the poles from -90 to 90, and horizontally around the equator between 0 and 360.

The Aster dataset above provides named tiles that span the globe in neat one degree increments. The following code uses the primitive number as the basis for reading the correct files from the dataset, mapping a single image to a single prim in the grid. This gives us a 2D world map made from the Aster image set, easily spherized to become a globe.

However, the above range of possible integer coordinates would make a set of 64800 image tiles (180*360), and we only have 23k(ish). Why? Because tiles that are all ocean (and therefore contain no useful height information) are omitted from the set.

At this point I encountered the need to develop a strategy that compensated for missing image tiles. Now, this can be left to the renderer, at least in the case of Arnold. The default behavior when encountering missing textures is to halt the render, but it's behaviour that can be turned off. With the missing texture colour be set to black (it defaults to red, which for displacement is not good), the height information for missing images is made good.

But I wanted to be more explicit in handling missing textures, rather than having Arnold unnecessarily scanning the filesystem for tiles that do not exist on every render. I was also keen to avoid switching off otherwise useful default behaviors, and in general faffing aronud with multiple things in multiple contexts where possible. I felt more comfortable feeding Arnold a single black image that represented any image that was missing from the Aster set.

This PythonSOP takes care of that. It looks for files, and sets the image tile to a catch all black image for a prim that has no corresponding file on disk. The result of this operation I cached in order to avoid repeated and potentially expensive filesystem access, reducing the task to a cheap per point attribute lookup.

Okay. With missing textures dealt with, on we go.

Next, .tx file generation.

Pulling images into a render for the first time will cause Arnold's autotx utility create .tx textures from the source images. From the artists' perspective this is a pause, making the first render with Arnold slower than subsequant renders when all .tx files have been created.

When asking Arnold to consume hundreds (if not thousands - though I never got that far) of textures and multiple GBs of data, this first render can be painfully slow. TX generation was therefore a task that I wanted to treat as a preprocess, rather than relying on autotx, a horribly long wait, and an seemingly frozen IPR window.

So I used a useful commandline tool (oiiotool) to generate the .tx files away from Houdini. Specifically I used a version of the oiiotool utility that ships with Houdini (hoiio.exe). This can be found at the following path:

The following Windows shell script is the one I used to convert the input images. It checks to see if the .tx file exists, and if not it will create the file using hoiio. It's important that the output matches the autotx output, otherwise Arnold will attempt to overwrite the existing.tx file.

It's also worth noting that I created a simple TOPs graph ino rder to distribute .tx generation (even locally it ran faster than using the comnadline - presumably TOPs does a better job of saturating CPU by completing multiple conversions concurrently). However, I quickly ran into a bug when asking TOPs to maintain many thousands of workitems over a slow office network (odd timeout issues that didn't occur locally). Given that the majority of my .tx files were created after a weekend away from my workstation, I didn't persue any optimisation further. I had my .tx files and the job was done; though I have included the TOPs network in the HIP for general interest.

WITH ALL THAT DONE, I started to explore methods of using the images in the project... But unfortunately it was at this point that my research concluded.

So I didn't actually get beyond a few preliminary tests at using the Aster data, either for displacement in SOPs or at rendertime, although I did start to think about how I might use the data while avoiding the possibility of Arnold picking up the entire set of 22k tiles with each rendered frame.

Strategies broadly involved separating a high resolution region of interest away from the rest of the planet based on proximity to camera, or perhaps geometry that fell inside a certain portion of the frame. This way I could target the high resolution data to where it was needed, and a cheaper region away from the area of interest. Could the region of interest be sent to comp independatly of the background earht surface, or could the seams be blended to avoid the second render? Stuff to explore, and some excercises for the reader.


PLANETARY CLOUDS

More specifically, clouds being blown around the planet at speed.

Another friendly colleague showed me this:

https://earth.nullschool.net/

Those are some useful force vectors. Even if they're not, at least they can shut down any talk of "That looks fake, you shiuld use real world data". This is real world data, and everyone can bog off.

Inspecting the source of the site took me to the project's Github page, which in turn led me to the following archive of weather information. There's an overwhelming amount of stuff available, but after staring at it for a bit I managed to fathom what I needed:

https://nomads.ncep.noaa.gov/

GRB files are gridded binary files... I've know idea what that means, but I do know that it's not JOSN, XML, or any other easily comprehensible text format. So step one was to translate the GRB files into something I could feed into Houdini. 

It turns out that there is another commandline tool available to convert GRB files to JSON. Unfortunately, this one is Linux only and don't come as a compiled binary. Bums. The following is whizzed over at speed because it relies on knowledge of a few tools that are beyond the scope of these notes... 

I had to spin up a Linux virtual machine using Vagrant. With vagrant you can effectively create a computer within a computer, for me it enabled the running of a Linux box inside a Windows box. Don't be deterred. It's easy to get to grips with and HowTos abound online. Give it a go, Vagrant up!

On compilation of the ECC tools, I found a very handy guide that removed any need for thinking about what I was doing:

https://gist.github.com/MHBalsmeier/a01ad4e07ecf467c90fad2ac7719844a

With all that done, I was able to convert the entirely opaque GRB file, into a very transparent JSON file to read inside HOUDINI.

I then had to remap that data, as before with the DEM data, but using a different set of tricks given the different input. The result of that, wind vectors in HOUDINI!

Again, it was at this ponit that I stepped away from the research, so I'd only given a modest amount of thought to the next stage in production. Rendering...

I thought of a few different approaches to rendering planetary clouds, and in the end I selected remapping a 2D pyro sim around a sphere. From a simulation POV this is far faster than anything in 3D, ultimately we're skimming the outer surface from a potentially expensive 3D pyro sim, and a fewq early tests at rasterising points didn't produce anything that i wanted to continue with. Mapping a rectangle to a sphere obviously comes with distortion at the poles, but for the shots considered in proudction this didn't matter. Lucky me. So much better to fly through simulation and put up with the constraints of wrapping the result around a sphere. I'm sure you're smarter than me, and will take this further