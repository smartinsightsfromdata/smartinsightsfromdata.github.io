---
title: Complete choropleth development workflow in R (Part 1)
date: 2015-04-21
---

I've attended the last DataKind Data Dive in Leeds.

Once more, we had to confront the issue of producing attractive choropleth from shapefiles, in a short time.

I thought to document the entire workflow here, as every time I struggle to remember how to do this, going back to my poorly commented old files.

This workflow is the result of many hours of efforts consulting many blogs and stackoverflow posts.  I am grateful for all the help received but I am unable to recognise everyone.  Perhaps the best way to say thank you is to give back to the community in the form of this post.

### The process

The process is farily simple, but includes many different steps and many tools and/or libraries, hence it is important to know what you are doing (or follow the steps precisely!).

The steps are

1. Get shapefiles
2. Get the shapefiles of the area of interest
3. Select the shapefile (and re-project if necessary)
4. Prepare the data
5. Simplify the shapefile and save it to geojson
6. Depending on the libraries you want to use, convert to topojson
7. Select the banding cuts and how to implement them
8. Test the choropleth in a browser

First step of course is to get adequate shapefiles.

### Get shapefiles

The easiest thing, especially if you need to build a choropleth with census or generally public data, is to get the shapefile from the [ONS Open Geography Portal](https://geoportal.statistics.gov.uk/geoportal/catalog/main/home.page).

Before this though it would be appropriate decide which level of detail the choropleth will have.  We are spolied for choices, but to keep this simple I will simply select to download shapefiles at the LSOA (lower layer super output areas) level.  This [page](http://www.ons.gov.uk/ons/guide-method/geography/beginner-s-guide/census/super-output-areas--soas-/index.html) in the ONS website will guide you on the difference between different output areas.  Of course more "traditional" choies like wards etc. are also available.
The shapefiles come in two flavours:
- clipped, basically excluding shores and potentially some river islands
- non-clipped

My personal choice is usually to select clipped shapefiles.

At this stage we will select the files.  For England this is approximately a folder of 550MB. 

### Select the shapefiles

First of all, what is a shapefile?  It is a term usually referring to a combination of polygons coordinates, Ids, optional spatial indexes and data that refer to a specific geographical area.
There are many standards for shapefiles.  We will mention only two standards:

- ESRI shapefiles. They in turn come split into different files addressing some of the characteristics discussed above (e.g. polygons, data etc.).

- geojson. Json is a great, concise standard apt to group polygons and data. 

As there is a visual element in all this, it is important (in my experience) to see what is it that you get.  For this I normally use QGIS.

On the other hand, as this is a R workflow, in this case I'll stick to R.

The following code will allow us to see what is in the shapefiles (as normally you download ESRI shapefiles from the ONS portal I'll use this format).

Please note that the packages used are rgeos and rgdal.  The complexities and pitfalls in properly installing these libraries are out of scope of this brief tutorial.  Let me know if is a topic I should cover more in detail.
{% highlight R %}
library(rgdal)
library(rgeos)
#
dns <- "/Users/e/Dropbox/dev/DevLib/MyGISLib/Lower_layer_super_output_areas_(E+W)_2011_Boundaries_(Full_Clipped)"
ogrInfo(dns, "LSOA_2011_EW")
  Source: "/Users/e/Dropbox/dev/DevLib/MyGISLib/Lower_layer_super_output_areas_(E+W)_2011_Boundaries_(Full_Clipped)", layer: "LSOA_2011_EW"
  Driver: ESRI Shapefile number of rows 36008 
  Feature type: wkbPolygon with 2 dimensions
  Extent: (82672 5337.9) - (655604.7 657534.1)
  CRS: +proj=tmerc +lat_0=49 +lon_0=-2 +k=0.9996012717 +x_0=400000 +y_0=-100000 +ellps=airy +units=m +no_defs  
  LDID: 0 
  Number of fields: 2 
      name type length typeName
  1 LSOA11CD    4      9   String
  2 LSOA11NM    4    254   String
{% endhighlight %}
Now let's interpret the above information.
As discussed above, these are the fully clipped shapefiles for the LSOA areas in England and Wales (E+W).
There are a total of 36008 polygons in England and Wales.
The extent is the total "box" area covered by England and Wales.

Here we encounter the first "real" issue: the coordinate reference system (CRS) follow the British National Grid (i.e. OSGB36). This is great **but** often enough is not the standard normally used for choropleth.

If you want to superimpose your choropleth layer onto an existing map, and you want to avoid **any** major licensing issue, you need to use open source maps like OpenStreetMap.  These maps tend to use WGS84 as a coordinate reference system.  Inclusing Google map and Bing, which are not free and have clear use restrictions, follow a variation of WGS84 (often referred to EPSG: 3785).
Incidentally EPSG: 3785 itself may create erros of up to 20Km in UK alone, as explained in this clear [article](https://alastaira.wordpress.com/2011/01/23/the-google-maps-bing-maps-spherical-mercator-projection/).

Next part we will cover selecting and re-projection (i.e. how to fix the above issue and use the proper CRS).



