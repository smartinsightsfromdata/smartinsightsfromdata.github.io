---
title: Complete choropleth dev. workflow in R (Part 4)
date: 2015-04-25
---

(this is a continuation from Part 3)

#### The leaflet library

As mentioned in Part 3, the leaflet libaries is developed by RStudio (a guarantee of quality!) and it is based on the **best** JavaScript mapping library: [leaflet.js](http://leafletjs.com/) by Vladimir Agafonkin. It is Leaflet is an open-source JavaScript library for mobile-friendly interactive maps: only 33 KB of JS!

The R library leaflet is undergoing a major revamp as we speak. in order to get (some of) the latest features, I've used the following code to install it in my laptop:

```{r}
devtools::install_github("rstudio/leaflet", ref="feature/color-legend")
```

I expect that the "feature/color-legend" branch will be merged in master in a matter of days and specify a branch will no longer be necessary.

This is the "core" code used to render the map (inspired by the work of Yihui Xie and Joe Cheng at RStudio).

```R
#------------------------------------------------------------------
# As we may want to play with different colours palettes or binning types, variables etc., 
# a function may be useful

myMap = function(shape,szoom, pal, area,vals,Pops, ...) {
  mbox <- bbox(shape)
  cLong <- round((mbox["x","min"]+mbox["x","max"])/2,4)
  cLat <- round((mbox["y","min"]+mbox["y","max"])/2,4)
  #
  leaflet(data = shape)  %>% setView(cLong, cLat, szoom) %>% addTiles() %>%
    addPolygons(fillColor = pal(vals), 
                weight = 2,
                opacity = 1,
                color = 'white',
                dashArray = '3',
                fillOpacity = .5, 
                popup = sprintf("<strong>%s</strong><br/>%g%%  of %g", area, vals,Pops)
    ) %>%
    addLegend(pal = pal, values = vals, ...)
}
```

A few comments:

- The leaflet function call components are piped using the great magrittr package.
- leaflet will by default use openstreetmap (this is the addTiles() portion).  Other types of maps, usually open source,  can be used.  See [here](http://leaflet-extras.github.io/leaflet-providers/preview/index.html).
- It is useful to centre the map. The easiest way to do it is to get the "box" of the overall shape, which are the minimum and maximum long / lat coordinates of the geojson shapes we are using, and calculate the centre.
- We discussed popups in Part 3. Here I show an implementation with sprintf.  It can be made more complex and display data on multiple lines (and it could as well become a character specified as function parameter).

#### Binning & Colour Palettes

As discussed we build a choropleth to diplay numerical variables, either continuous or discrete.
As we want to colour the different areas in relationship to a choosen variable, we need to find a way to bin the data in the number of cuts we want to use for the map.

On the other hand the number of cuts (or bins) depends from the available number of colours in the colour palette we have choosen.

A good start is to choose the colour palette (but it is a chicken and egg argument).

The absolute reference here is Chintya Brewer and her amazing work at PennState. I strongly recommend to spend time at the [site she created](http://colorbrewer2.org/), trying the different palettes and understanding the nature of data we need to display (sequential, divergent or qualitative) and whether we need to cater for colour blind people. 

She has also inspired the creation of the R package RColorBrewer.

The colour palette I have choosen is 'YlOrRd'.  Again I recommend to go onto Chintya website and experiment on different palettes. 

Please consider that the maximum number of colours for sequential data is 9, so we will have to use 9 bins or less.

leaflet offers the following binning functions:

- colorNumeric is a is a simple linear mapping from continuous numeric data
#' to an interpolated palette.
- colorBin maps continuous numeric data performing binning based on value through the base::cut function. From R help: "cut divides the range of x into intervals and codes the values in x according to which interval they fall. The leftmost interval corresponds to level one, the next leftmost to level two and so on."
- colorQuantile is similar to colorBin but uses the base::quantile function. From R help: "The generic function quantile produces sample quantiles corresponding to the given probabilities. The smallest observation corresponds to a probability of 0 and the largest to a probability of 1."

I have decided to display unemployment percentage figures in Leeds LSOAs, so the function colorBin is the most appropriate. I will use 5 cuts.

```R
pal = colorBin('YlOrRd', leedsShape@data$Unemployed_, 5)
myMap(leedsShape, szoom=11,pal,vals= leedsShape@data$Unemployed_,area= leedsShape@data$Name,
         Pops= leedsShape@data$Population, title = '% Unemployment')
```

I published the entire code of this part [here](http://rpubs.com/enzoma/leedsGeo02), where you can also navigate the map (zoom etc.).

Below is a static pic of the resulting choropleth:

![plot](/images/Rplot.png)


This concludes Part 4 of the tutorial.

