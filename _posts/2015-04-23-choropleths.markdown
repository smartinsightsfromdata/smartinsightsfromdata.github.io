---
title: Complete choropleth dev. workflow in R (Part 3)
date: 2015-04-23
---

(this is a continuation from Part 2)

#### Features of choropleth libraries

Some choropleth implementation may offer one or both of the following features:

- Hover-over. Basically this makes available the information of which polygon the mouse / cursor is on to the client / server.  In turn this is exploited for displaying information relative to the area enclosed by the polygon.
- Mouse-click. If a polygon is clicked, the information is made available to the client / server and again this can be used to display different types of information.
- Legend may shows how the different areas are doing, based on different colour schemes and cuts.
- Popup legend may be associated to hover-over or mouse-clicks events.
- Availability of pre-packaged geographical information to make the loading of a particular shapefile unnecessary. Usually the area involved is the US: therefore the feature is not really appreciated in the rest of the world.

Other considerations point to increase levels of sophistication, moving from simple choropleth to the ability to:

- Add icons and points in different layers.
- Add drill down from a larger region to a smaller region displaying different colours.

Choropleth are rendered through different media, e.g. markdown documents, html web pages, as components of a shiny app etc. Different style of interaction may be expected.  This will influence our choice of library

While I always keep an eye out for new, better things (and welcome any indication or suggestion), for the rest of this tutorial I will focus on the following two libraries:

- `leaflet` from RStudio, integrating the famous JS library `leaflet` developed by Vladimir Agafonkin.
- `rdatamaps` from smartinsightsfromdata (yours truly), integrating the D3-based `datamap` library (this will allow also to cover the topojson format).

#### Introduction to Data Preparation

If we are building an analytical application, is very important to address what do do with the data.  
A choropleth can display well **one** type of information at a time, usually a numerical variable (continuous or discrete).

If we want to make available lots of data in our choropleth the user will have to have ways to choose which dataset (or which feature) to see. A way to handle multiple datasets or data items is to divide the data into the following categories:

- Data that will directly be used to "colour" the choropleth map
- Data displayed in popups depending from user action.
- Data to possibly display on another layer with dots (maybe using 2 features like size and colours) or different icons.
- data displayed in tables associated to display in the page beside the actual choropleth, providing additional detail.

#### Local vs. remote

In association with the above the design of our analytical app will have to determine which data will be in the client at any one time, and which data download to the client on demand.

Just recently a new concept is emerging for analytical applications: whether the map creation should be **local** or **remote**.  See Joe Cheng (the SW architect that has designed the shiny framework in R) [post](https://github.com/rstudio/leaflet/pull/60) introducing this interesting concept.

Basically a choropleth is made of a map plus a number of other features, first and foremost the data associated to it.
Should the choropleth just be **local**, and destroyed every time something is changed, or should it be **remote**, with the different elements added and subtracted as driven by server logic?

Clearly I can imagine plenty of cases where it could be beneficial a remote map creation to manage flexibly and dynamically many types of analysis.  More on this later.


#### Prepare the data

At the moment we do not have interesting data to use for the choropleth.  Let's fix that.

to start with we can use the LSOA data available in the [Leeds Observatory](http://observatory.leeds.gov.uk/dataviews/view?viewId=235). I've downloaded in csv format the ```LSOA subset: Leeds```

```
library(rgdal)
library(rgeos)
library(data.table)
options(warn=-1)
#--------------------------------------------------------------------
# Read the LSOA polygons (already simplified) geojson

dsn <- ("./data/leedsWgs84.geojson")
leedsShape <- readOGR(dsn=dsn, layer="OGRGeoJSON", stringsAsFactors=FALSE)
# check on number of rows

nrow(leedsShape)
#--------------------------------------------------------------------
# Read the csv table (I *love* to use the fread function)

dt_ <- fread("./data/leedsLSOA.csv")
#check on number of rows

nrow(dt_)
#--------------------------------------------------------------------
# Column names cleaning and selection

colnames(dt_)[1:2] <- c( "LSOA","Name")
colnames(dt_)[5] <- "Population"
colnames(dt_)[378:379] <-  c("Unemployed","Employed")
dt_ <- dt_[,.(LSOA, Name, Unemployed=as.numeric(Unemployed), 
      Employed=as.numeric(Employed), Population= as.numeric(Population))][,
      Unemployed_:= round(Unemployed/Population*100,2)]

#--------------------------------------------------------------------
# do we have an identical number of matching elements?

identical(dt_[,LSOA] ,leedsShape@data$LSOA11CD)

# no! - only 440: LSOA are changing to keep the pace with population increase /decrease:

length(intersect(leedsShape@data$LSOA11CD,dt_[,LSOA] ))

#--------------------------------------------------------------------
# This is to "merge" the data portion of the geojson to the data we will display
# this technique just "works" preserving order and matching
# other techniques may give errors like "row.names of data and Polygons IDs do not match"

leedsShape@data <- data.frame(leedsShape@data, dt_[match(leedsShape@data[, "LSOA11CD"], dt_[, LSOA]),])

leedsShape@data$LSOA11NM <- leedsShape@data$LSOA <- NULL  # eliminate duplicated or unused columns
```

The comments to the code should be self-explanatory.  The main issue are the 42 missing or mismatched polygons.  For the time being we will keep them as missing.

This concludes Part 3 of the tutorial.

