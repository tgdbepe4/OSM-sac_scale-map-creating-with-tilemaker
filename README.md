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


