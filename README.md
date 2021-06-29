# OSM-sac_scale-map-creating-with-tilemaker

## Demo
https://sacscale.bergi-it-consulting.ch/index.html

## Introduction
The idea is to create a map which shows the difficulties of hiking pathes and tracks for the Open Street Map source. The attribute "sac_scale" can be used in OSM to declare the difficulty of hiking pathes, https://wiki.openstreetmap.org/wiki/DE:Key:sac_scale. At the moment the usage of this attribute is far a way to be compete but in higher alpine regions there many pathes available with it. 

## Tool for creation of vector tiles
https://github.com/systemed/tilemaker has created an excelent tool to create vector tiles. The tool allows to use as input an OSM pbf file and create from an mbtiles file which can be use from different tile servers. For this project only the tileserver-php software was used. 



## Making of with Switzerland OSM pbf file

On any linux system:

* Clone from https://github.com/systemed/tilemaker and follow the installation instructions.
* Clone https://github.com/maptiler/tileserver-php and follow up the installation instructions.
* wget http://download.geofabrik.de/europe/switzerland-latest.osm.pbf
* tilemaker --input switzerland-latest.osm.pbf  --output sacscale.mbtiles  --config resources/config-openmaptiles.json  --process resources/process-openmaptiles.lua
* cp switzerland.mbtiles /var/www/html/tileserver-php/
* service apache2 stop  
* service apache2 start
* load URL/index.html into your browser

##Files
* config-openmaptiles.json (config-openmaptiles.json.ori)
    * In this file is configured how detailed the render process goes. And for each layer when the layer is visible (zoom level).

* process-openmaptiles.lua (process-openmaptiles.lua .ori)
    * This file describes which OSM tag is rendered and which attribute are added.

* sacscale_style.json
    * In this file the look and feel is described, e.E. if a river is persent in which colour and if the river name is shown. This file has to be part of the tileserver-php configuration. Without this file there is nothing to see! Check this blog https://blog.kleunen.nl/blog/tilemaker-generate-map

* Tileserver for testing
To serve your tiles use the demonstration server:

cd server
# ruby server.rb /path/to/your/output.mbtiles
# You can now navigate to http://localhost:8080/ and see your map!

(If you don't already have them, you'll need to install Ruby and the required gems to run the demonstration server. On Ubuntu, for example, sudo apt install sqlite3 libsqlite3-dev ruby ruby-dev and then sudo gem install sqlite3 cgi glug rack.)