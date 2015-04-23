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

![plot](/images/LeedsLSOA.png)

Now we will change the projection, moving from OSGB36 to WSG84.

```
leedsCoor <- CRS("+proj=longlat +datum=WGS84")
newLeedsShape <- spTransform(leedsST, leedsCoor)
```

We could re-plot the shapefiles, but probably at this stage we wouldn't notice a major difference.

#### Simplify the shapefile and save it to geojson

This step is very simple.  Basically a polygon contains numerous nodes, i.e. points: every nook and cranny of the area of reference is detailed.  This is too much detail for a choropleth. The size of the Leeds shapefile we have obtained is around 16Mb. This would make it difficult to show our choropleth on a mobile phone.

The solution is simple: it is possible to simplify the polygons reducing the number of nodes. We use the well established Ramer–Douglas–Peucker algorithm (RDP) to reduce the number of points in a curve that is approximated by a series of points. 
We need to be careful though: too much simplifications and the polygons will start to "open up": basically they will not appear attached to each other anymore and holes will start to manifest. This is where it is convenient to use tools like QGIS to try the best tolerance coefficient for the simplification.
I will be satisfied with a reduction to ~10% of the original size. 
The simplification uses ```gSimplify``` from the ```rgdal``` package.

```R
# gSimplify strips the data portion away - it needs to be saved first
>df <- newLeedsShape@data
#
>newLeedsSimp <- gSimplify(newLeedsShape, 0.00005, topologyPreserve=TRUE)
# now we can recombine the two portions of the SpatialPolygonsDataFrame
>newLeedsSimpShape <- SpatialPolygonsDataFrame(newLeedsSimp,data=df)
# eventually we can save the file to geojson
>writeOGR(newLeedsSimpShape,'./data/newLeedsSimp.geojson','newLeedsSimpShape', driver='GeoJSON',check_exists = FALSE)
```

The original shapefile has been seriously simplified and re-projected. What if we have made some mistake along the way?

A very simple, but very useful trick we can use to check on our geojson is to load it in a gist in github.

Github will render the shapefile over a zoomable map of the are: we can see directly the quality of what we have done so far.

{% gist 303edd54995ba6fcdd09 %}

Another interesting characteristic of our map is that clicking on a polygon will display its Code Name and Name. We will come back to exploit this feature later.

#### A note of caution

The method of using github is great.  During the process of writing this simple tutorial I've discovered few interesting things I had forgotten or I didn't know.
- geojson standard is actually pased on a WGS84 projection. If you want to be really specific, "urn:ogc:def:crs:OGC:1.3:CRS84".  This info appears in the json file as CRS.
- rgdal / gdal automatically re-project the shapefiles when saving to geojson if it is not already in the right projection.
- Github uses openstreetmap (the main open source map) to render geojson in their gists. Currently github supports only urn:ogc:def:crs:OGC:1.3:CRS84 (e.g. WGS84).
- As the Ordnance Survey site puts it, it is a myth to believe that is possible to re-project exactly from one CRS to another with a simple algorith (and even a "complex" one like the seven point Helmert transformation gives some residual errors).

All of the above notwithstanding, I've got a real surprise when I saw that that the geojson produced with this pipeline does not superimpose with the openstreetmap on github but it is sligthly shifted, which normally does not happen when I use QGIS (which uses gdal as well).  I'm investigating and report on this issue asap.

This concludes Part 2 of this series.  Next part will adress the data preparation.









