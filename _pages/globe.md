A while ago I completed a project that involved the development of high-resolution world maps. These maps used publically available DEM and weather data, and these notes discuss some strategies I developed to deal with that data.

It's worth noting that some of the strategies are somewhat specific to Arnold, which we used in production; though I've included them here because similar hurdles would need to be overcome regardless of chosen render.

Digital Elevation Model DEM

Some colleagues exposed me to large DEM data sets, essentially huge clumps of images containing height data to use in the geometric or rendertime displacement of geometry. The following dataset offered level of quality that satisfied the resolutino of the project and the proximity of the camera to the earth's surface:

https://search.earthdata.nasa.gov/search/granules?p=C1711961296-LPCLOUD&pg[0][v]=f&pg[0][gsk]=-start_date&fi=ASTER&tl=1703889672.142!3!!

But. That's 22k images and about 400GB of data. Yowza.

REMAPPING
A lot of the work required mapping one grid of values to another. This is broadly trivial, but each remapping requires a unique set of steps to transform values into something meaningful.

Latitude and longitude are a set of coordinates that start at the intersection between the prime meridian and the equator, extending vertically between the poles from -90 to 90, and horizontally around the equator between 0 and 360.

For all the mults and shifts, all the following code does is read the filenames from the dataset, and massage them into a format that associates one image to each prim in the grid. We now have the set of images mapped to a 2D map, and this map is easily sphereized to become the displaced globe.

The Aster dataset provides named tiles that span the globe in neat one degree increments. Given the above range of possible integer coordinates, this would make a full set that spans 64800 tiles (180*360). HOWEVER, tiles that are all ocean (and therefore contains no useful height information) are omitted from the set, reducing the set to 22k tiles.

So at this point I encountered the need to develop a strategy that compensated for missing image tiles. This can obviously be left to the renderer, at least in the case of Arnold vs missing textures. The default behavior is when encountering missing textures is to halt the render, behaving that can be turned off, and then the resulting error colour be set to black (it defaults to red, which for displacement is not good).

This is all very well, but I wanted to be more explicit in handling missing textures, rather than having Arnold potentially scanning the filesystem for hundreds of tiles that do not exist onevery render. I felt much omre comfortable feeding Arnold a single black image that represented any image that was missing from the Aster data.

This Python sop takes care of that. It looks on the file system for missing files, and resets the CMAP property of the tile to a catch all texture. The result of this operation can be cached to disk in order to avoid repeated and potentially expensive filesystem access, reducing the task to a relatively cheap per point attribute lookup.

Okay. With that dealt with, on we go.

Pulling textures into Arnold for the first time will generate .tx textures, which makes the first render with Arnold slower than with subsequant renders. When asking Arnold to consume hundreds of textures and multiple Gbs of data, this first render can be painfully slow. It's was therefore a task that I wanted to treat as a preprocess, rather than relying on autotx, a horribly long wait, and a frozen IPR window.

So I used commandline tools to generate the .tx files away from Houdini. Specifically I used a version of the oiiotool utility that ships with Houdini (hoiio.exe). This can be found at the following path:

The following Windows shell script is the one I used to convert the textures. It checks to see if the .tx fiel exists, and if not it will created using hoiio.

It's also worth noting that I did get a TOPs process going that I wanted to distributed to our farm, which even locally ran faster than using the comnadline (presumably it does a better job of saturating CPU by running multiple conversions concurrently), but I ran into a bug asking TOPs to maintain many thousands of workitems over a network, and given that after a weekend away my .tx textures had been generated, that I didn't persue TOPs any further; though I have included it in the HIP for interest.

WITH ALL THAT DONE, I started to explore methods of using the remapped in the project... at which point the project took a different steer that unfortunartely negated the need for high-resolution data. Shucks.

So I didn't actually get beyond a few preliminary tests at using the Aster data in renders, although I did start to think about how I might use the data while avoiding the possibility of Arnold picking up the entire set of data with each rendered frame. Strategies broadly involved separating a patch of high resolutino terrain based on proximity to camera, or geometry that fell within a certain portion of the frame. This way I could have a high-resolutino expanesave region, and a cheaper region away from the area of interest. Excercise for the reader?


WIND VECTORS
Discuss solutions. 2D speed, 3D with colliders, 2D pops to attrib to volume.
NOMAD
https://nomads.ncep.noaa.gov/
https://earth.nullschool.net/

Install guide:
https://gist.github.com/MHBalsmeier/a01ad4e07ecf467c90fad2ac7719844a

VAGRANTCOMPILE SOFTWRAR