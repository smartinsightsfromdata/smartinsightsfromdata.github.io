---
title: Complete choropleth dev. workflow in R (Part 2)
date: 2015-04-22
---
(this is a continuation from Part 1)

### Select the shapefile (and re-project if necessary)

I reproduce here part of the code used in Part 1. Please note that further details of the structure of an ESRI shapefile are covred [here](http://www.gdal.org/drv_shapefile.html)

{% highlight R %}
# code omitted
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

The "data" part of the shapefile comes at the end:

{% highlight R %}
  1 LSOA11CD    4      9   String
  2 LSOA11NM    4    254   String
{% endhighlight %}

As the ONS documentation clarifies, the LSOA for the 2011 census come in:
- Code names of 9 characters
- Names of up to 254 characters

Strangely you often find that the codes are similar or identical. In any case we will use the LSOA11CD codes to subset the shapefiles.






