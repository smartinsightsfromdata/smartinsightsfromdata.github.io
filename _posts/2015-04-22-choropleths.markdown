---
title: Complete choropleth dev. workflow in R (Part 2)
date: 2015-04-22
---
(this is a continuation from Part 1)

#### Select the shapefile (and re-project if necessary)

A description of the structure of ESRI shapefiles and their treatment by the gdal library are covered [here](http://www.gdal.org/drv_shapefile.html).

Let's read the actual shapefiles. The rgdal package reads them into a spatial object: a SpatialPolygonsDataFrame, to recognise their dual nature (polygons + data).

```R
> ewAll <- readOGR(dsn=dns, layer="LSOA_2011_EW",  stringsAsFactors=FALSE)
OGR data source with driver: ESRI Shapefile 
  Source: "/Users/e/Dropbox/dev/DevLib/MyGISLib/Lower_layer_super_output_areas_(E+W)_2011_Boundaries_(Full_Clipped)", layer: "LSOA_2011_EW"
  with 36008 features
  It has 2 fields
# It is helpful understand the class of the ewAll object
> class(ewAll)
  [1] "SpatialPolygonsDataFrame"
  attr(,"package")
  [1] "sp"
```

In Part 1 we have already encountered the "data" part of the shapefile:

```
1 LSOA11CD    4      9   String
2 LSOA11NM    4    254   String
```

As the ONS documentation clarifies, the LSOA for the 2011 census come in:
- Code names of 9 characters
- Names of up to 254 characters

Let's see what sort of data we have inside the shapefiles.

```R
> head(ewAll@data)
   LSOA11CD         LSOA11NM
0 E01030056 Mid Suffolk 006B
1 E01030057 Mid Suffolk 007B
2 E01030054 Mid Suffolk 002B
3 E01030055 Mid Suffolk 002C
4 E01030052 Mid Suffolk 001A
5 E01030053 Mid Suffolk 003A
```

Therefore the plan of attack is clear: 
- If we need to subset the shapefile, we should be able to find the area(s) of interest searching the extended names.
- If we will need to link data to the shape we will use the code names (much more about this later).

As the Data Dive I've attended was about Leeds, I will select the shapefiles referring to Leeds.

```R
>leedsST <- ewAll[grepl( "Leeds", ewAll$LSOA11NM),]
> nrow(leedsST)
[1] 482
#
plot(leedsST)
```

In this case I know that all the LSOA shapes in Leeds have "Leeds" in the name. In some other case it will be more complex, but often is possible to find lookup tables to ease the task. This gives us 482 polygons for all the Leeds area. As the average population ofr a LSOA is 1,500 inhabitants, a quick check (482*1,500) will tell us that the order of magnitude is correct. Good!

We can now plot the shapefile. Before would have been too much for RStudio, my IDE of choice (and even tools like QGIS struggle a bit for the sheer number of polygons to display).










