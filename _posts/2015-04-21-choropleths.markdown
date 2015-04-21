---
title: Complete choropleth development workflow in R
date: 2015-04-21
---

I've attended the last DataKind Data Dive in Leeds.

Once more, we had to confront the issue of producing attractive choropleth from shapefiles, in a short time.

I thought to document the entire workflow here, as every time I struggle to remember how to do this, going back to my poorly commented old files.

This workflow is the result of many hours of efforts consulting many blogs and stackoverflow posts.  I am grateful for all the help received but I am unable to recognise everyone.  Perhaps the best way to say thank you is to give back to the community in the form of this post.

###The process

The process is farily simple, but includes many different steps and many tools and/or libraries, hence it is important to know what you are doing (or follow the steps precisely!).

The steps are

1. Get shapefiles
2. Select the shapefiles of the area of interest
3. Prepare the data
4. Simplify the shapefile and save it to geojson
5. Depending on the libraries you want to use, convert to topojson
6. Select the banding cuts and how to implement them
7. Test the choropleth in a browser

First step of course is to get adequate shapefiles.

###Get shapefiles

The easiest thing, especially if you need to build a choropleth with census or generally public data, is to get the shapefile from the [ONS Open Geography Portal](https://geoportal.statistics.gov.uk/geoportal/catalog/main/home.page).

Before this though it would be appropriate decide which level of detail the choropleth will have.  We are spolied for choices, but to keep this simple I will simply select to download shapefiles at the LSOA (lower layer super output areas) level.  This [page](http://www.ons.gov.uk/ons/guide-method/geography/beginner-s-guide/census/super-output-areas--soas-/index.html) in the ONS website will guide you on the difference between different output areas.  Of course more "traditional" choies like wards etc. are also available.
The shapefiles come in two flavours:
- clipped, basically excluding shores and potentially some river islands
- non-clipped

My personal choice is usually to select clipped shapefiles.

At this stage we will select the files.  For England this is approximately a folder of 550MB. 


Some code:

{% highlight R linenos=table %}
   a <- 3
   c <- lapply(a, typeof)
   print(c)
{% endhighlight %}


