---
title: Complete choropleth dev. workflow in R (Part 2)
draft
---

date: 2015-04-22
(this is a continuation from Part 2)

#### Intro to choropleth libraries

Before getting to the details of data preparation, we need to put our strategic thinking hats and make some design decision.  In order to appreciate what I'm going to say, a bit of explanation first.

Some of the features of modern choropleth implementation offer one or both of the following features:
- hover-over. Basically this makes available to the client / server when the cursor is above a polygon.  In turn this is exploited for displaying information realtive to the area enclosed by the polygon.
- mouse-click. If a polygon is clicked, the information is made available to the client / server and again this can be used to display different types of information.

A choropleth may also offer different types of legends:
- A legend that shows how the different areas are doing, based on different colour schemes
- Possibly a popup legend to associate to the above click and hover events.

Other consideration point to increase levels of sofistication, moving from simple choropleth to the ability to:
- Ability to add icons and points in different layers.
- Ability to drill down a choropleth and potentially display different, smaller regions.

Another feature, not exacly valued outside of the US, i if the library contains already geographical information to make the loading of a particular shapefile unnecessary.

There are many libriaries in R that offer some or many of the above functionalities out-of-the-box or with add-ons.

While I always keep an eye out for new, better things (and welcome any indication or suggestion), for the rest of this tutorial I will focus on the following two libraries:
- leaflet from RStudio, integrating the famous JS library leaflet).
- rdatamaps from smartinsightsfromdata (yours truly), integrating the D3-based datamap library (this will allow also to cover the topojson format).

#### Working with the leaflet library

At the time of writing this post, the leaflet library is undergoing a heavvy, very welcome, functional uplift, so for the time being, I will focus on the key functions, to revist them at a later stage to integrate more advanced functionalities.


#### Prepare the data

At the moment we do not have interesting data to use for the chopleth.  Let's fix that.

to start with we can use the LSOA data available in the [Leeds Observatory](http://observatory.leeds.gov.uk/dataviews/view?viewId=235). I've downloaded in csv format the ```LSOA subset: Leeds```
