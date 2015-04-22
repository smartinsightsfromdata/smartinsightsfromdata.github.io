---
title: Complete choropleth dev. workflow in R (Part 2)
date: 2015-04-22
---
(this is a continuation from Part 1)

### Select the shapefile (and re-project if necessary)

A detailed description of the structure of ESRI shapefiles and their treatment by the gdal library are covered [here](http://www.gdal.org/drv_shapefile.html)

Let's read the actual shapefiles. The rgdal package reads them into an object defines as SpatialPolygonsDataFrame, to recognise their dual nature (polygons + data).
{% highlight R %}
> ewAll <- readOGR(dsn=dns, layer="LSOA_2011_EW",  stringsAsFactors=FALSE)
OGR data source with driver: ESRI Shapefile 
  Source: "/Users/e/Dropbox/dev/DevLib/MyGISLib/Lower_layer_super_output_areas_(E+W)_2011_Boundaries_(Full_Clipped)", layer: "LSOA_2011_EW"
  with 36008 features
  It has 2 fields
# It is helpful understand the class of the ewAll object
> class(ewAll)
  [1] "SpatialPolygonsDataFrame"
  attr(,"package")
  [1] "sp
{% endhighlight %}

In Part 1 we have already encountered the "data" part of the shapefile:

{% highlight R %}
  1 LSOA11CD    4      9   String
  2 LSOA11NM    4    254   String
{% endhighlight %}

As the ONS documentation clarifies, the LSOA for the 2011 census come in:
- Code names of 9 characters
- Names of up to 254 characters

Strangely you often find that the codes are similar or identical. In any case we will use the LSOA11CD codes to subset the shapefiles.






