---
title: Complete choropleth dev. workflow in R (Part 2)
draft
---

date: 2015-04-22
(this is a continuation from Part 2)

#### Intro to choropleth libraries

Before getting to the details of data preparation, we need to put our strategic thinking hats and make some design decision.  In order to appreciate what I'm going to say, a bit of explanation first.

Some of the features of modern choropleth implementation offer one or both of the following features:
- hover-over. Basically this makes available to the client / server when the cursor is above a polygon.  In turn this is exploited for displaying information realtive to the polygon.
- mouse-click. If a polygon is clicked, the information is made available to the client / server and again this can be used to display different types of information.


#### Prepare the data
