

# OSM-sac_scale-map-creating-with-tilemaker

## Demo
https://sacscale.bergi-it-consulting.ch/index.html
#### 3D view of Zurich
![image](https://user-images.githubusercontent.com/46052442/125194385-5dde2500-e251-11eb-84c2-542569fcd133.png)
#### 2D view of Zurich and the sac_scale pathes
![image](https://user-images.githubusercontent.com/46052442/125194454-aeee1900-e251-11eb-95ce-082b485209ef.png)


## Introduction
The idea is to create a map which shows the difficulties of hiking pathes and tracks for the Open Street Map source. The attribute "sac_scale" can be used in OSM to declare the difficulty of hiking pathes, https://wiki.openstreetmap.org/wiki/DE:Key:sac_scale. At the moment the usage of this attribute is far a way to be compete but in higher alpine regions many pathes are available with it. 

## Tool for creation of vector tiles
https://github.com/systemed/tilemaker has created an excelent tool to create vector tiles. The tool allows to use as input an OSM pbf file and create from an mbtiles file which can be used on different tile servers. For this project only the tileserver-php server software was used. 

## Making of with Switzerland OSM pbf file

On any linux system as an example:

* Clone from https://github.com/systemed/tilemaker and follow the installation instructions.
* Clone https://github.com/maptiler/tileserver-php and follow up the installation instructions.
* wget http://download.geofabrik.de/europe/switzerland-latest.osm.pbf as an example
* run tilemaker --input switzerland-latest.osm.pbf  --output sacscale.mbtiles  --config resources/config-openmaptiles.json  --process resources/process-openmaptiles.lua
* cp switzerland.mbtiles /var/www/html/tileserver-php/
* service apache2 stop  
* service apache2 start
* load URL/index.html into your browser

## Files
### config-openmaptiles.json (config-openmaptiles.json.ori)
In this file it is configured how detailed the render process goes. And for each layer when the layer is visible (zoom level).
#### Additonal shapefiles in config
It is also possible to add own self created shapefiles to the config file. 
##### Shapefile for cliffs
Tilemaker shows cliffs as polygons instead of line. This has the impact that a cliff is show as a filled region. Fortunaltely tilemaker allows to add Shapefile as additional imput. 
Performed in this way:
* installation of osmosis software and QGIS
* On windows run osmosis\bin> .\osmosis.bat --rbf switzerland-latest.osm.pbf --way-key-value keyValueList=natural.cliff --wb cliffs_switzerland_4326.osm.pbf
* Load the file cliffs_switzerland_4326.osm.pbf into QGIS, select only lines for input
* Export the layer in 4326 projection as shapefile, cliffs_switzerland_4326.shp
* Place the generated file into a sub folder of the tilemaker folder
* Add a line "cliff": 			 { "minzoom": 6,  "maxzoom": 14, "source": "data/cliffs_switzerland_4326.shp"}," near line 28 into file config-openmaptiles.json
* Add some lines into switzerland_style.json:

      {
      "id": "cliff",
      "type": "line",
      "source": "openmaptiles",
      "source-layer": "cliff",
      "minzoom": 13,
      "maxzoom": 24,
      "filter": [
        "all",
        ["==", "$type", "LineString"],
        ["in", "class", "ocean"]
      ],
      "layout": {"line-join": "round", "visibility": "visible"},
      "paint": {
        "line-color": "rgba(0, 0, 0, 1)",
        "line-width": {"base": 1.2, "stops": [[14, 1.5], [20, 10]]},
        "line-opacity": 1
      }
      },
 
 Now you be able to see the cliffs as lines

### process-openmaptiles.lua (process-openmaptiles.lua .ori)
This file describes which OSM tag is rendered and which attribute are added.
#### Addings to lua
##### landuseKeys and landcoverKeys
Expanding the values, near line 128:

    landuseKeys     = Set { "school", "university", "kindergarten", "college", "library", "hospital",
                        "railway", "cemetery", "military", "residential", "commercial", "industrial",
                        "retail", "stadium", "pitch", "playground", "theme_park", "bus_station", "zoo",
					    "farmyard" }
    landcoverKeys   = { wood="wood", forest="wood",
                    wetland="wetland",
                    beach="sand", sand="sand",
                    farmland="farmland", farm="farmland", orchard="farmland", vineyard="farmland", plant_nursery="farmland",
                    glacier="ice", ice_shelf="ice",
					bare_rock="bare_rock", rock="rock", cliff="cliff",
					scree="scree",
                    grassland="grass", grass="grass", meadow="grass", allotments="grass", park="grass", village_green="grass", recreation_ground="grass", garden="grass", golf_course="grass" }
##### Support of 3D buildings
This is a very nice feature. Do the fact that not all buildings have a hight tags which declares the hight of the building but in some case a building-levels tag. In such a case the value of the buildding-level is taken and calculate for this value an approximate hight.
Add near line 30:

    -- The height of one floor, in meters

    BUILDING_FLOOR_HEIGHT = 3.66
    
Add the end of the lua file:   

	function SetBuildingHeightAttributes(way)
	local height = tonumber(way:Find("height"), 10)
	local minHeight = tonumber(way:Find("min_height"), 10)
	local levels = tonumber(way:Find("building:levels"), 10)
	local minLevel = tonumber(way:Find("building:min_level"), 10)local renderHeight = BUILDING_FLOOR_HEIGHT
	if height or levels then
		renderHeight = height or (levels * BUILDING_FLOOR_HEIGHT)
	end
	local renderMinHeight = 0
	if minHeight or minLevel then
		renderMinHeight = minHeight or (minLevel * BUILDING_FLOOR_HEIGHT)
	end

	-- Fix upside-down buildings
	if renderHeight < renderMinHeight then
		renderHeight = renderHeight + renderMinHeight
	end

	way:AttributeNumeric("render_height", renderHeight)
	way:AttributeNumeric("render_min_height", renderMinHeight)
    end



### switzerland_style.json
In this file the look and feel is described, e.E. if a river is persent in which colour and if the river name is shown. This file has to be part of the tileserver-php configuration. Without this file there is nothing to see! Check this blog https://blog.kleunen.nl/blog/tilemaker-generate-map.
The original file was at many places modified, extended and re ordered. Please check the complete file for the source.
#### 2D and 3D
Near at the end of the "*style.json" add this to make 3D buildings visible:

    
    {
      "id": "building-3d",
      "type": "fill-extrusion",
      "source": "openmaptiles",
      "source-layer": "building",
      "minzoom": 14,
      "layout": {"visibility": "visible"},
      "paint": {
        "fill-extrusion-color": "rgba(203, 198, 198, 1)",
        "fill-extrusion-height": {
          "property": "render_height",
          "type": "identity"
        },
        "fill-extrusion-base": {
          "property": "render_min_height",
          "type": "identity"
        },
        "fill-extrusion-opacity": 0.9
      }
    },
If the fill-extrusion-opacity is lower 0.9 then you see the steets going through the buildings which is not very nice. For reason the building-3d part should be near by the end of the file.
An other problem viwe 3D buildings is that house number are always place on the ground. This has the effect, that are shown at a strange place. For this reason there are two HTML files used, one for 2D with house numbers and one for 3D without house numbers.

## Tileserver for testing
To serve your tiles use the demonstration server:

    cd server*
    ruby server.rb /path/to/your/switzerland-latest.mbtiles

You can now navigate to http://localhost:8080/ and see your map!  This map is very helpful because it shows from each obkject the attributes as well.

(If you don't already have them, you'll need to install Ruby and the required gems to run the demonstration server. On Ubuntu, for example, sudo apt install sqlite3 libsqlite3-dev ruby ruby-dev and then sudo gem install sqlite3 cgi glug rack.)

## HTML file on server
To control the 3D in the browser add after style: 'switzerland_style_3D.json'

    style: 'switzerland_style_3D.json',
    pitch: 60, // pitch in degrees
    bearing: 20, // bearing in degrees
    maxBounds: bounds 

        <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>OSM "sac_scale" map</title>
    <meta name="viewport" content="initial-scale=1,maximum-scale=1,user-scalable=no">
    <link href="https://api.mapbox.com/mapbox-gl-js/v2.3.1/mapbox-gl.css" rel="stylesheet">
    <script src="https://api.mapbox.com/mapbox-gl-js/v2.3.1/mapbox-gl.js"></script>
    <style>
    body { margin: 0; padding: 0; }
    #map { position: absolute; top: 0; bottom: 0; width: 100%; }
    </style>
    </head>
    <body>
    <div id="map"></div>
    <script>
    var bounds = [ [5.893, 45.748], [10.6318, 47.8541] ];
    	mapboxgl.accessToken = 'pk.eyJ1IjoidGdkYmVwZTQiLCJhIjoiY2trYmJxcWZmMGN2cjJucWp2NTZuYnRwMiJ9.6_xPbpP13tLTVNUk2cMq0Q';
    var map = new mapboxgl.Map({
    container: 'map',
    style: 'mapbox://styles/mapbox/light-v10',
    
      center: [ (bounds[0][0] + bounds[1][0]) / 2, (bounds[0][1] + bounds[1][1]) / 2],
      center: [8.5, 47.35], // starting position [lng, lat],
        zoom: 14,
      minZoom: 5,
      style: 'switzerland_style_3D.json',
      pitch: 60, // pitch in degrees
      bearing: 20, // bearing in degrees
      maxBounds: bounds 
    });
     
    map.on('load', function () {
    map.addSource('mapbox-terrain', {
    type: 'vector',
    // Use any Mapbox-hosted tileset using its tileset id.
    // Learn more about where to find a tileset id:
    // https://docs.mapbox.com/help/glossary/tileset-id/
    url: 'mapbox://mapbox.mapbox-terrain-v2'
    });
    map.addLayer({
    'id': 'terrain-data',
    'type': 'line',
    'source': 'mapbox-terrain',
    'source-layer': 'contour',
    'layout': {
    'line-join': 'round',
    'line-cap': 'round'
    },
    'paint': {
    'line-color': '#0387C5',
    'line-width': 0.5
    }
    });
    });
    
    map.addControl(new mapboxgl.NavigationControl());
    map.addControl(new mapboxgl.FullscreenControl());
    // Add geolocate control to the map.
    map.addControl(
    new mapboxgl.GeolocateControl({
    positionOptions: {
    enableHighAccuracy: true
    },
    trackUserLocation: true
    })
    );
    </script>
    <!-- <div id="osm">©<a href="http://www.openstreetmap.org">OpenStreetMap</a>
      und <a href="http://www.openstreetmap.org/copyright">Mitwirkende</a>,
      <a href="http://creativecommons.org/licenses/by-sa/2.0/deed.de">CC-BY-SA</a>
    </div> -->
    </body>
    </html>
    <!-- Copy
    © MapboxTermsPrivacySecurity -->
    
  ## Experiencs
  ### Performance
  I tried to render europe on a 64 GB Ram, 6 CPU computer with a additional swap space of 50 GB. This could work but a gave it up after some days of prcessing. The --store option save some memory but the performance is not better.
  Result: With a 64 GB computer the maximum pbf file is about 4 GBytes file size. 
  ### Merging
  The --merge option allows to merge a new rendered pbf file into an existing mbtiles file. This works in princip very good. I ran several pdb files in on shot via a shell program. This is not recommended because when failing somethin on the way all in the merged mbtiles file is lost.  
  
