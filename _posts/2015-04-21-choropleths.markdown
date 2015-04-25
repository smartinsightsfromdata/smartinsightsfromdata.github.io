---
title: Complete choropleth dev. workflow in R (Part 1)
date: 2015-04-21
---

I attended the last DataKind Data Dive in Leeds.

Once more, we had to confront the issue of producing attractive choropleths from shapefiles, in a short time.

I thought to document the entire complete workflow here, as every time I struggle to remember how to go about to do some of the steps, going back to my poorly commented old files, or I start to use too many tools.

This workflow is the result of many hours of efforts consulting many blogs and stackoverflow posts.  I am grateful for all the help received but I am unable to recognise every contribution.  Perhaps the best way to say thank you is to give back to the community in the form of this post.

#### The process

The process is farily simple, but includes many different steps and many R libraries, hence it is important to know what you are doing (or follow the steps precisely!).

The steps are

1. Get shapefiles
2. Get the shapefiles of the area of interest
3. Select the shapefile (and re-project if necessary)
4. Simplify the shapefile and save it to geojson
5. Intro to choropleth libraries
6. Prepare the data (e.g. banding cuts etc.)
7. The leaflet library
8. The rdatamaps library
9. Lesson learned

First step of course is to get adequate shapefiles.

#### Get shapefiles

The easiest thing, especially if you need to build a choropleth with census data or generally open data, is to get the shapefile from some official source. In the UK the [ONS Open Geography Portal](https://geoportal.statistics.gov.uk/geoportal/catalog/main/home.page) offers a great service (I understand developed also with the help of the Ordnance Survey).

Before this though it would be appropriate decide which level of detail the choropleth will have (e.g. from continents and major countries to post code analysis).  This mostly depends from the type of analysis you need to do.

The following code will assume to use UK local data, but the examples and the code is easily extensible to any area or country.
Becasue of the standard set by the ONS in the UK in general we are spolied for choices, but to keep this simple I will select to download shapefiles at the LSOA (lower layer super output areas) level.  This [page](http://www.ons.gov.uk/ons/guide-method/geography/beginner-s-guide/census/super-output-areas--soas-/index.html) in the ONS website will guide you on the difference between different output areas.  Of course more "traditional" choies like wards etc. are also available.
The shapefiles come in two flavours:
- clipped, basically excluding shores and potentially some river islands
- non-clipped

My personal choice and in general for business use it is usual to select clipped shapefiles.

At this stage we will select and download the files.  For England and Wales this is approximately a folder of 550MB. 

#### Select the shapefiles

First of all, what is a shapefile?  It is a term usually referring to a combination of polygons coordinates, Ids, optional spatial indexes and data that refer to a specific geographical area.
There are many standards for shapefiles.  For the sake of building a choropleth I will mention only the following two standards:

- ESRI shapefiles. They in turn come split into different files addressing some of the characteristics discussed above (e.g. polygons, data etc.).

- geojson. Json is a great, concise standard apt to group polygons and data (among other things!). 

As there is a visual element in all this, it is important (in my experience) to see what is it that you get.  For this I normally use QGIS, which allows also to select the area visually.

On the other hand, as this is a R workflow, this time I'll stick to R.

The following code will allow us to explore what is in the shapefiles (as the ESRI shapefiles are the most common - and are also used by ONS I'll use this format).

Please note that the R packages used are rgeos and rgdal.  The complexities and pitfalls in properly installing these libraries and their underlying libraries (they require in turn the installation of gdal and proj4) are out of scope of this brief tutorial.  Having lost so many hours over time to install them in different generations of OS and computer I sympathize with everyone that has gone through similar challenges. Let me know if is a topic I should cover more in detail.

```R
>library(rgdal)
>library(rgeos)
#
>dns <- "/Users/e/Dropbox/dev/DevLib/MyGISLib/Lower_layer_super_output_areas_(E+W)_2011_Boundaries_(Full_Clipped)"

>ogrInfo(dns, "LSOA_2011_EW_BFC")
 Source: "/Users/e/Dropbox/dev/DevLib/MyGISLib/Lower_layer_super_output_areas_(E+W)_2011_Boundaries_(Full_Clipped)_V2", layer: "LSOA_2011_EW_BFC_V2"
Driver: ESRI Shapefile number of rows 34753 
Feature type: wkbPolygon with 2 dimensions
Extent: (82672 5337.9) - (655604.7 657534.1)
CRS: +proj=tmerc +lat_0=49 +lon_0=-2 +k=0.9996012717 +x_0=400000 +y_0=-100000 +datum=OSGB36 +units=m +no_defs  
LDID: 87 
Number of fields: 3 
       name type length typeName
1  LSOA11CD    4      9   String
2  LSOA11NM    4    254   String
3 LSOA11NMW    4    254   String
```
Now let's interpret the above information provided by rgdal.

As discussed above, these are the fully clipped shapefiles for the LSOA areas in England and Wales (E+W).
There are a total of 34753 polygons in England and Wales.
The "extent" are the coordinates of the total "box" including England and Wales.

Here we encounter the first "real" issue: the coordinate reference system (CRS) follow the British National Grid (i.e. OSGB36). 
It is a transverse Mercator projection with an origin (the "true" origin) at 49° N, 2° W (an offshore point in the English Channel which lies between the island of Jersey and the French port of St. Malo), based on the Airy ellipsoid, as well as a "false" origin to eliminate negative numbers.

This is very interesting and great **but** often enough OSGB36 is **not** the standard normally used for choropleth.

If you want to superimpose your choropleth layer onto an existing map, and you want to avoid **any**  licensing issue, you need to use open source maps like OpenStreetMap.  
These maps tend to use WGS84 as a coordinate reference system.  Also other well known maps, including Google map and Bing, which are not free and have different types of use restrictions, follow a variation of WGS84 (often referred to EPSG: 3785).
Incidentally EPSG: 3785 itself may create errors of up to 20Km in UK alone, as explained in this clear [article](https://alastaira.wordpress.com/2011/01/23/the-google-maps-bing-maps-spherical-mercator-projection/).

If we would use shapefiles with a CRS "a" to superimpose onto a map with a CRS "b", the result will probably be horrendous.

This means that we will need to reproject our shapefiles to WGS84 to be able to superimpose correctly the choropleth polygons to the underlying map layer.

Next part  will cover the selection of the sahefiles for the area of interest and the shapefiles re-projection (i.e. how to fix the above issue and use the CRS of choice).



