
# OSM-sac_scale-map-creating-with-tilemaker

## Demo
https://sacscale.bergi-it-consulting.ch/index.html

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
The original file was at many places modified, extended and re ordered. Please the complete file for the source.

## Tileserver for testing
To serve your tiles use the demonstration server:

    cd server*
    ruby server.rb /path/to/your/switzerland-latest.mbtiles

You can now navigate to http://localhost:8080/ and see your map!  This map is very helpful because it shows from each obkject the attributes as well.

(If you don't already have them, you'll need to install Ruby and the required gems to run the demonstration server. On Ubuntu, for example, sudo apt install sqlite3 libsqlite3-dev ruby ruby-dev and then sudo gem install sqlite3 cgi glug rack.)
