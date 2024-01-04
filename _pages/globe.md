A while ago I began to research a project based around high-resolution world maps. Dramatic camera work was planned, with the camera plummeting from outerspace to a few kilometers away from the earth's surface.

I looked at solutions for 3D topographic detail, and planetary clouds.

Unfortauntely it's a project that never came to fruition, so we never reached the final frame, but enough work was completed to warrent some production notes. It's also rare that encounter a contribution that isn't tedious social media banter, and with that perhaps some of the following will be of use to a future someone...

It's worth noting that some of the notes are somewhat specific to Arnold, being the render that would be used in prodction; though I've included them here because similar hurdles might need to be overcome regardless of renderer.


Digital Elevation Model DEM

A helpful colleague pointed me toward a DEM data sets that are availble from the Nasa website. They are essentially large archives of topographic image tiles sourced from satelites scanning the globe at regular intervals. These can be used to drive displacement. One particular dataset offered a level of quality that satisfied the production resolution given the proximity of the camera to the earth's surface:

https://search.earthdata.nasa.gov/search/granules?p=C1711961296-LPCLOUD&pg[0][v]=f&pg[0][gsk]=-start_date&fi=ASTER&tl=1703889672.142!3!!

What a resource! But also, we've just shy of 23k images and around 400GB of data. Yowza.

REMAPPING
A lot of the work required mapping one set of values (the image filenames) to another (a grid of primitives). This is broadly trivial, but any remapping requires a unique set of steps to transform values into something meaningful.

Latitude and longitude are a set of coordinates that zero at the intersection between the prime meridian and the equator, extending vertically between the poles from -90 to 90, and horizontally around the equator between 0 and 360.

The Aster dataset above provides named tiles that span the globe in neat one degree increments. The following code uses the primitive number as the basis for reading the filenames from the dataset, mapping a single image to a single prim in the grid. This gives us a 2D world map made from the Aster image set, easily spherized to become a globe.

However, the above range of possible integer coordinates would make a set of 64800 image tiles (180*360), and we inly have 22k(ish). This is because tiles that are all ocean (and therefore contains no useful height information) are omitted from the set.

At this point I encountered the need to develop a strategy that compensated for missing image tiles. This can be left to the renderer, at least in the case of Arnold. The default behavior is when encountering missing textures is to halt the render, but it's behaviour that can be turned off. With the missing texture colour be set to black (it defaults to red, which for displacement is not good), the height information across the globe is functional.

But I wanted to be more explicit in handling missing textures, rather than having Arnold unnecessarily scanning the filesystem for hundreds of tiles that do not exist on very render. I felt more comfortable feeding Arnold a single black image that represented any image that was missing from the Aster set, rather than switching off oterhwise useful default behaviours.

This Python sop takes care of that. It looks on the file system for missing files, and sets the image tile of a prim that has no corresponding texture on disk to a catch all black image. The result of this operation I cached to disk in order to avoid repeated and potentially expensive filesystem access, reducing the task to a cheap per point attribute lookup.

Okay. With missing textures dealt with, on we go.

Pulling textures into a render for the first time will cause Arnold's autotx utility to pause while it creates .tx textures. This makes the first render with Arnold slower than subsequant renders, when all .tx files have been created. When asking Arnold to consume hundreds (if not thousands - though I never got that far) of textures and multiple gigs of data, this first render can be painfully slow. TX generation was therefore a task that I wanted to treat as a preprocess, rather than relying on autotx, a horribly long wait, and an uncomfortably frozen IPR window.

So I used a useful commandline tool (oiiotool) to generate the .tx files away from Houdini. Specifically I used a version of the oiiotool utility that ships with Houdini (hoiio.exe). This can be found at the following path:

The following Windows shell script is the one I used to convert the textures. It checks to see if the .tx file exists, and if not it will created using hoiio. The output matches

It's also worth noting that I did create a simple TOPs graph working that could therefore be distributed (even locally it ran faster than using the comnadline - presumably it does a better job of saturating CPU by completing multiple conversions concurrently), but I quickly ran into a bug asking TOPs to maintain many thousands of workitems over a slow network (odd timeout issues that didn't occur locally), and given that after a weekend away from my desk while it crunched through the Aster data my .tx textures had been generated, that I didn't persue any optimisation any further. I had my .tx files and that was that; though I have included the TOPs network in the HIP for general interest.

WITH ALL THAT DONE, I started to explore methods of using the images in the project... But unfortunately it was at this point that my research concluded.

So I didn't actually get beyond a few preliminary tests at using the Aster data in renders, although I did start to think about how I might use the data while avoiding the possibility of Arnold picking up the entire set of 22k tiles with each rendered frame. Strategies broadly involved separating a high resolution region of interst away from the rest of the planet based on proximity to camera, or perhaps geometry that fell inside a certain portion of the frame. This way I could target the high resolution data to where it was needed, and a cheaper region away from the area of interest. I didn't get very thought, so I'll have to leave that as an excercise for the reader?


WIND VECTORS

Another aspect of the (project was to be the rendering of realistic cloud data). Again, a source of publicically avoiding real world weather data proved of interest. This site was offered to me as a useful resource:

Inspecting the source odf this website took me to the proect's Github page, which in turn led me to the following archive of weather information. It's a little overwhelmeing, but after staring at it for a bit I managed to grasp what I need:

GRB files are gridded binary files... I've know idea eaxactly what that means, but I d know that it's not JOSN, XML, or any other easily interpreted text format. So step one was to translate the GRB files into something I could read into Houdini. 

It turns out that there are more commandline tool savailable to convert these GRB fiels to JSON. Unforateunyl, this tools are Linux only and don't come as a compiled binary. To cut a long story short, becaseu the detail is way beyondf the scope of a few crib notes, I had to spin up a Linux vertual machine using a piece of software that I was familiar with from a former involvemtn with web development. Vagrant. Using vagrant you can effectively create a computer within a computer, for me it enabled the running of a Linux box inside a windows box, and a box that I can use and distory at the drop of a hat.

On compilation of the ECC tools, I found a very handy guide that removed any thinking:

With all that done, I was able to convert the entirely opaquwe GRB file, into a very transparent JSON file to read inside HOUDINI.

I then had to remap that data, as before with the DEM data, but using a fresh set of tricks given the different underring data. The result of that, wind vector data in HOUDINI!

On rendering. I thought of a few different approaches to rendering planetary clouds, and in the end I selected remapping a 2D pyro solution to a sphere. From a simulation POV this is far faster than anything in 3D, and ultimately we're skimming the outer surface from a potentially expensive 3D pyro sim, and while it comes with obvious distortion aroundf the poles, for the shots considered in proudction this didn't matter to uys. So much better to fly through. So not a universal solution, but a fast one that worked wewll within prodeuction constraints.

Discuss solutions. 2D speed, 3D with colliders, 2D pops to attrib to volume.
NOMAD
https://nomads.ncep.noaa.gov/
https://earth.nullschool.net/

Install guide:
https://gist.github.com/MHBalsmeier/a01ad4e07ecf467c90fad2ac7719844a

VAGRANTCOMPILE SOFTWRAR