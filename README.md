# Create custom hiking maps using OSM
HRP, OSM maps, Maperitive, hikingmaps

I recently completed a solo throughhike of the Haute Randonèe Pyrènèene (HRP) from the Atlantic to the Mediterranean Sea, taking about 45 days. This was undoubtedly one of the best trips I have ever done. Today I don't want to give a trip report, but want to focus on one aspect that took me quite some preparation time: Maps.

While there are official maps of the french and spanish pyrenees available (some info [here](https://lukaszsupergan.com/hrp-haute-route-pyreneenne-gear-trail-map-guide/)), covering the entire mountain chain would mean carrying 12 maps, which in addition to signifying quite a bit of weight, would also mean a substantial lightening of my purse. Many people nowadays rely completely on digital maps, using phones or dedicated GPS devices. However, I dislike the idea of relying completely on technilogy that could easily fail in cold or wet conditions. I really love navigating with paper maps and compass, I feel that I have a better overview of where I am, and where I am going when using paper instead of squiting at a tiny display. In addition, paper maps are great for planning out the next days and getting a feel for the terrain that is coming.

The unavailability of paper maps that I could buy at a reasonable price meant that I had a chance to use my computer science degree for once for something practical: to generate my own maps.

## Overview
According to my usual perfectionistic standards, I embarked on this project with a humble goal: to create the perfect personalised maps for my hiking trip. They should fulfill the following criterions:
- style like a standard hiking map, showing height lines and terrain
- show custom GPS tracks and points of interest (huts, water points, resupply points, etc)
- final printed maps should have minimal weight with maximal information

To my great surprise and delight I managed to tick all three boxes. The final maps looked similar to 1:25000 hiking maps I usually use, but were printed with high dpi on a scale of 1:50000 on A3 paper. They were ordered in such a way that I could throw them away after finishing with one map, lightning my load considerably towards the end. They contained the HRP, GR10, and GR11 GPS tracks, and specifically highlighted water sources, campsites and resupply points.

I used [Maperitive](http://maperitive.net), a great open steet map (OSM) rendering tool, and [hikingmap](https://github.com/roelderickx/hikingmap), a python project that allows the creation of strip maps allong a GPS track.  Additonal tools I used were [QMapShack](https://github.com/Maproom/qmapshack/wiki) and [Inkscape](https://inkscape.org). In the remainder of this post I describe my process for creating the maps to hopefully allow you to create your perfect maps for your next hiking adventure as well.

## Preparation
I put all software and data in one folder, which I will call `$RENDER_OSM` in the following.

### Software
For creating your own maps, download and install the following in `$RENDER_OSM`:
* [Maperitive](http://maperitive.net)
* hikingmap and hm-render-maperitive: I modified some functions a bit, find my fork of the original hikingmap project [here](https://github.com/hspitzer/hikingmap) and of the hm-render-maperitive renderer [here](https://github.com/hspitzer/hm-render-maperitive)
* my custom scripts [render_osm](https://github.com/hspitzer/render-osm)

### Data
Ultimately, I used the `download_osm_overpass` option of Maperitive, allowing to download OSM data on the fly while rendering the maps. However, it is also possible to download data beforehand:
* see instructions [here](https://wiki.openstreetmap.org/wiki/Downloading_data#Small_amounts_of_data). E.g., export this [map](https://www.openstreetmap.org/export#map=7/43.129/1.725)
* divide data in smaller stages, save in `$RENDER_OSM/render_osm/data/section01.osm`
* when using this data, it is also necessary to divide GPX tracks into per-section tracks and save them to `$RENDER_OSM/render_osm/data/GPX_section01.gpx`

### DEM (Digital Elevation Model)
A very important feature of good hiking maps are elevation lines. For this I used the following:
- download required tiles from [viewfinderpanoramas](http://www.viewfinderpanoramas.org/Coverage%20map%20viewfinderpanoramas_org3.htm)
- copy hgt files to `$RENDER_OSM/Maperitive/Cache/Rasters/SRTM3`
- to display elevation lines, need to run `generate-contours` in Maperitive

### GPS Tracks
I wanted to display the HRP, the GR10 and the GR11, and waypoint information about water, camping and resupply options.
I used the HRP tracks from the [HRP pocket guide](https://whiteburnswanderings.wordpress.com/2018/06/04/the-hrp-a-pocket-guide/), waypoint information from [LesBlogsDeFranck](https://lesblogsdefranck.jimdofree.com/pyrénées/hrp-haute-route-des-pyrénées/), huts from [here](https://www.pyrenees-refuges.com/#8/42.446/0.360), and GR10 and GR11 tracks from links provided [here](http://www.lasenda.net/trekking-the-pyrenees-gr10-gr11-or-hrp-pyrenean-haute-route-a-short-guide-to-the-differences/). 

I processed the GPX files as follows:
* add `<cmt>` tags to each GPS track, denoting if HRP, GR10, GR11. This is for choosing the right color to display it as later on
* join tracks in one GPX file and save to `$RENDER_OSM/render_osm/data/GPX_all.gpx`
* add `<cmt>` tag to each waypoint, indicating the type of the waypoint, to display it with the right icon later on. 

I used python to add the `<cmt>` tags. For other modifications and merging of GPX tracks, I used QMapShack.

You can find the final gpx files [here](https://github.com/hspitzer/render-osm/tree/main/data)

## Create custom Maperitive rendering rules
A great feature of Maperitive is that it allows you to exactly define how you would like to render your maps. There are a couple of styles available [in the internet](https://wiki.openstreetmap.org/wiki/Maperitive/SampleRenderings), but for greatest flexibility, check out the [documentation](http://maperitive.net/docs/Rendering_Rules_Introduction.html) and learn how to create your own rules.

I extended the existing `Hiking` style to suit my needs. The main features that I added were the following:
- display difficulty information on top of each trail
- display name of waymarked trails
- custom icons for my custom waypoint information, using the `<cmt>` tag. Different colors for HRP, GR11, and GR10 tags. 
- nicer color scheme for printed maps (in my opinion at least, this is very subjective)
- textures for forest, grass, rock, scree
- highlight cliff lines

For adding textures, I found this [tool](http://www.imagico.de/map/jsdotpattern.php) that allows one to create aethetically pleasing random patterns. I used this to create patterns for the textures that I wanted to add, and modified the resulting svgs a bit to my liking. The final textures can be found [here](https://github.com/hspitzer/render-osm/tree/main/textures). Copy them to `$RENDER_OSM/Maperitive/Textures`.

My process for creating the ruleset was to iteratively change the rules and visualise the result. I also found [this tutorial](https://braincrunch.tumblr.com/post/4286086882/maperitive-python-scripting-introduction ) and [this example ruleset](https://projects.webvoss.de/2017/06/03/creating-the-perfect-hiking-map-for-germany-and-other-countries/#TutorialStart) very helpful for inspiration.

Maperitive commands that I used during this process were:
- run maperitive:
	- `/Library/Frameworks/Mono.framework/Versions/Current/bin/mono --arch=32 Maperitive/Maperitive.exe`
- load map:
	- `clear-map`
	- `load-source render_osm/data/pyrenees-1.osm`
- change rulesets:
	- `use-ruleset location=render_osm/scripts/HikingHannahHRP.mrules as-alias=hs`
	- `apply-ruleset`

My final Maperitive rules are available [here](https://github.com/hspitzer/render-osm/blob/main/scripts/HikingHannahHRP.mrules). Ensure they are available under `$RENDER_OSM/render_osm/scripts/HikingHannahHRP.mrules`. 


## Create strip maps along a GPX track
To export png files in A3 following along the main GPX tracks in `GPX_all.gpx`, I used hikingmap. This tool creates a python script with a series of maperitive commands to consecutively export png maps. 
The following command builds commands to export A3 svg maps with a margin of 16mm to be added later (8mm at each side) and 1cm overlap between maps. Run it from `$RENDER_OSM` folder:

```hikingmap --basename maps/svg/hrp. --pagewidth 28.1 --pageheight 40.4 --pageoverlap 1 --overview --gpx render_osm/data/GPX_all.gpx  -- hm_render_maperitive/hm_render_maperitive.py -d 300 -p 200 -f svg -c --download-data```

This creates files `hm-render-maperitive-xxxx.maperi.py`, `hikingmap_temp_overviewxyz.gpx` and `hikingmap_temp_waypointsxyz.gpx`.

I copied these files to `$RENDER_OSM/render_osm/hm-render-maperitive-hrp-large.maperi.py`, `$RENDER_OSM/render_osm/hikingmap_overview_large.gpx` and `$RENDER_OSM/render_osm/hikingmap_waypoints_large.gpx` and modified them as follows:
- remove downloading of OSM data for overview, and use `add-web-map` instead
- add page numbers to overview gpx track using `$RENDER_OSM/render_osm/scripts/modify_overview.ipynb`
- remove waypoints for GR10 and GR11, because I do not need kilometer markers there.

Note that I'm first exporting to SVG here, because font styles were incorrectly represented on the pngs. The ppi value is set to enable esport at zoom level 14 with map-scale 50000. For exporting as png, use dpi 200 and scale 2 (width=2338 scale=2).

Find the final created files [here](https://github.com/hspitzer/render-osm). Note that the absolute paths inside `hm-render-maperitive-hrp-large.maperi.py` need to modified to work on another workstation.

## Export maps with Maperitive and post-process
I wrote a small [script](https://github.com/hspitzer/render-osm/blob/main/scripts/HRP.mscript) to load all data and run the python script in Maperitive. Run this with

`/Library/Frameworks/Mono.framework/Versions/Current/bin/mono --arch=32 Maperitive/Maperitive.exe Scripts/HRP.mscript`

This generates SVG files. For creating pngs, use Inkscape. THe following exports pngs as 200 dpi (run from inside the `maps/svg` folder):
```
for f in hrp.*.svg; 
	do echo /Applications/Inkscape.app/Contents/Resources/bin/inkscape --export-png=/Users/hannah.spitzer/projects/personal/render_osm/maps/png/${f/svg/png}  --export-dpi=200 /Users/hannah.spitzer/projects/personal/render_osm/maps/svg/$f; 
done
```

Finally, we just need to add margins (8mm each side=131px) with Imagemagick and convert the final map to PDF of size a A3 (final resolution is 164 dots per cm). Run from `maps/png` folder:
```
for f in hrp.*.png; do 
	echo $f
	test=`convert $f -format "%[fx:(w/h>1)?1:0]" info:`
	if [ $test -eq 1 ]; then
		convert $f -rotate 270 -bordercolor white -border 131 -page A3 -units PixelsPerCentimeter -density 164 /Users/hannah.spitzer/projects/personal/render_osm/maps/pdf/${f/png/pdf}
	else
		convert $f -bordercolor white -border 131 -page A3 -units PixelsPerCentimeter -density 164 /Users/hannah.spitzer/projects/personal/render_osm/maps/pdf/${f/png/pdf}
	fi
done
```

Or, for printing, add "Anschnitt" of 3mm at each side, in addition to the A3 papersize:
```
for f in hrp_large.*.png; do 
	echo $f
	test=`convert $f -format "%[fx:(w/h>1)?1:0]" info:`
	if [ $test -eq 1 ]; then
		convert $f -rotate 270 -extent 6888x4871 -density 164 -units PixelsPerCentimeter -gravity center -bordercolor white -border 49 /Users/hannah.spitzer/projects/personal/render_osm/maps/pdf/${f/png/pdf}
	else
		convert $f -extent 6888x4871 -density 164 -units PixelsPerCentimeter -gravity center -bordercolor white -border 49 /Users/hannah.spitzer/projects/personal/render_osm/maps/pdf/${f/png/pdf}
	fi
done
```

## Conclusion
This is it! Easy, right? To be honest, it took me a couple of weeks of playing with Maperitive and the hikingmap export options to understand the rulesets and the relationships between ppi, dpi, resolution, etc. 
I hope this collection of scripts can help you to go through this process faster and create your perfect hiking maps. 

I was truly happy with the finished maps. Even though the contour lines were not perfect in several places, the maps never lead me astray and I only needed to use my phone a few times in the basque country where many horse and cow paths made navigating with a map very challenging.

PICTURE
