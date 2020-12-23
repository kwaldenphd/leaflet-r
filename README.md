# Making Maps R

<a href="http://creativecommons.org/licenses/by-nc/4.0/" rel="license"><img style="border-width: 0;" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" alt="Creative Commons License" /></a>
This tutorial is licensed under a <a href="http://creativecommons.org/licenses/by-nc/4.0/" rel="license">Creative Commons Attribution-NonCommercial 4.0 International License</a>.

## Lab Goals

This tutorial covers how to create interactive maps in R, focusing on two types of visualizations:
- Maps with markers, which are icons displayed at particular geographic locations
- Choropleth maps, sometimes called heat maps, where geographic units such as countries, states, or watersheds are colored according to a particular variable

[Link to lab overview video in Panopto (ND users only)](https://notredame.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=592c6b8f-d108-4905-a9d3-ac9a013f3859)

## Acknowledgements

This lab procedure is adapted from and based on Ryan Miller's ["Interactive and Animated Graphics Using plotly"](https://remiller1450.github.io/s230f19/plotly.html) (Fall 2019, Intro to Data Science STA 230 course, Grinnell College).

# Table of Contents

# What is `leaflet`?

There are many ways to create maps using R, but we will focus on leaflet, a popular open-source JavaScript library that is used by many professional organizations to create interactive maps.

[Link to lab overview video in Panopto (ND users only)](https://notredame.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=592c6b8f-d108-4905-a9d3-ac9a013f3859)

For more on leaflet, check out [the JavaScript library documentation](https://notredame.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=9cf6097a-1733-4882-a59c-ac9a013f8568).

For more on magrittr, check out [the package's tidyverse documentation](https://magrittr.tidyverse.org/).

# Data and Environment Setup

## Packages

We'll be using a number of packages in this lab.
- [`leaflet`](https://rstudio.github.io/leaflet/) "is one of the most popular open-source JavaScript libraries for interactive maps"
- [`geojsonio`](https://cran.r-project.org/web/packages/geojsonio/index.html) "convert[s] data to 'GeoJSON' or 'TopoJSON' from various R classes, including vectors, lists, data frames, shape files, and spatial classes"
- [`dplyr`](https://dplyr.tidyverse.org/) "is a grammar of data manipulation, providing a consistent set of verbs that help you solve the most common data manipulation challenges"
- [`magrittr`](https://magrittr.tidyverse.org/) "offers a set of operators which make your code readable," most notably through piping
- [`htmltools`](https://cran.r-project.org/web/packages/htmltools/index.html) provides "tools for HTML generation and output"

```R
# install.packages("leaflet")
# install.packages("geojsonio")
# install.packages("dplyr")
# install.packages("htmltools")
# install.packages("magrittr")

library(leaflet)    # The map-making package
library(geojsonio)  # A package for geographic and spatial data, requires the latest version of dplyr
library(dplyr)      # Used for data manipulation and merging
library(htmltools)  # Used for constructing map labels using HTML
library(magrittr)   # Used for piping
```

## Data

In this tutorial we will use two datasets: `quakes` and `happiness`

The “quakes” dataset, which contains the locations of 1000 seismic events occuring near the Fiji islands since 1964.

```R
data(quakes)
head(quakes)
```

The Happy Planet Index, a dataset of containing various measures of happiness and sustainability for countries around the globe.

```R
happiness <- read.csv('https://raw.githubusercontent.com/kwaldenphd/leaflet-r/main/HappyPlanet.csv', stringsAsFactors = FALSE)
head(happiness)
```

# Base Maps

Generally speaking, there are three steps to creating a map using leaflet:
- Creating a map widget by calling the leaflet function
- Adding layers to the map using functions like addTiles, addMarkers, and addPolygons
- Printing the map to display it

Layers are usually added by “piping” via the %>% operator. 

Put simply, this means passing the resulting object on the left of the %>% symbol as the first argument to the function on the right of the %>%, thereby chaining together a series of functions that each operate on the results of the prior step.

```R
## Step 1 call leaflet
map <- leaflet() %>%
          addTiles() ## Step 2 add a layer using %>%

## Step 3 print the map
map
```

Leaflet constructs a base map using map tiles, an approach popularized by google maps that is now used by nearly every interactive map online.

Calling addTiles() with no arguments will generate a base map using OpenStreetMap tiles. 

For most purposes OpenStreetMap tiles are all you’ll need.

However, the leaflet package also comes with dozens of different tile options created by third party providers and stored in an object named providers. 

Provider tile sets can be used to create a base map via the addProviderTiles function
```R
head(providers)  ## Providers contains a list of different tiles
```

```R
map <- leaflet() %>% 
          addProviderTiles(providers$Esri.WorldTopoMap)
map
```

# Maps with Markers

The simplest addition to base map is a marker, or identifiable point. 

Markers can be added to the base map using addMarkers function. 

At minimum a marker requires a longitude and latitude, but it is useful to also include labels, which display text when you hover over the marker, or popups, which display a dialog box when you click on the marker.

```R
fiji_reduced <- quakes[1:10,]

map <- leaflet(data = fiji_reduced) %>% 
        addTiles() %>% 
        addMarkers(lng = ~long, lat = ~lat, 
                   label = ~paste(depth, "meters"), 
                   popup = ~paste(mag, "Richter Magnitude", "<br/>", depth, "Depth"))

map
```

What is displayed by a popup or label is an interpretted HTML strings, meaning commands like “<br/>” can be used to separate the text onto a new line, and other HTML commands can be used to modify the text’s appearance.

Additionally, in the example above, you should notice the `∼` character that is used in front of the inputs to lng, lat, label, and popup. 

This tells addMarkers to look for those variable within the data object specified in the call to leaflet. 

So, in this example, leaflet looks for the variables “lat” and “lat” in fiji_reduced. 

Later in this tutorial we’ll see examples that don’t do this.

Question #1: 

Construct a base map using “Esri.OceanBasemap” tiles. Then, filter the quakes dataset to include only observations reported by more than 40 stations and plot the location of these earthquakes on your map with hoverable labels that display the number of stations reporting (ie: text like “47 Stations Reporting”).

## Marker Clusters

Adding a large number of markers to a map is computationally demanding, and it also makes the map difficult to read. 

To address these and other concerns, leaflet includes a suite of built in functions includingmarkerClusterOptions, which aggregates nearby markers into a group that will split when you click on it to zoom in.

```R
map <- leaflet(data = quakes) %>% 
        addTiles() %>% 
        addMarkers(lng = ~long, lat = ~lat,
                   label = ~paste(depth, "meters"),
                   clusterOptions = markerClusterOptions())

map
```

## Circle Markers

The appearance of markers can be changed in two ways. 

The most straightfoward is to use addCircleMarkers, which plots circular points at the marker locations. 

These markers have attributes like color, opacity, and fill that can be used to add information to the map. 

The addCircleMarkers function uses syntax similar to addMarkers, a few of its options are demonstrated in the example below:
```R
quakes$col <- ifelse(quakes$mag > 5, "red", "blue")

map <- leaflet(data = quakes) %>% 
        addTiles() %>% 
        addCircleMarkers(lng = ~long, lat = ~lat, 
                   label = ~paste(depth, "meters"),
                   color = ~col,
                   fillOpacity = 0.25,
                   stroke = FALSE)

map
```

Here, we color earthquakes with magnitudes over 5.0 red, while coloring less intense earthquakes blue. 

We use the argument fillOpacity = 0.25 to make the markers more transparent, allowing viewers to see relative concentrations, and we add the argument stroke = FALSE to remove the solid outline that by default surrounds circular markers.

## Custom Markers

Custom markers can be added to a map as icons. 

Icons are created from an image using its web url or local file path. 

The example below sets up two icons, a ferry icon and a skull with crossbones icon, using images from an online repository.

```R
  oceanIcons <- iconList(
    ferry = makeIcon(
      "http://globetrotterlife.org/blog/wp-content/uploads/leaflet-maps-marker-icons/ferry-18.png"),
    danger = makeIcon(
      "http://globetrotterlife.org/blog/wp-content/uploads/leaflet-maps-marker-icons/danger-24.png"))

iconid <- ifelse(quakes$mag < 5, "ferry", "danger")

leaflet(data = quakes[1:4,]) %>% addTiles() %>%
  addMarkers(~long, ~lat, icon = oceanIcons[iconid])
```

To use an icon, it must first be setup using makeIcon, and then stored in a special list using iconList before being passed to addMarkers.

In this example you should notice that we do not use a ∼ in front of oceanIcons. 

That is because oceanIcons exists outside of the quakes data frame.

Question #2

Construct a base map using the default map tiles. Then, using your filtered dataset (which contains earthquakes reported by more than 40 stations), construct a map displaying these earthquakes using clustered circle markers which are colored yellow if the earthquake was reported by more than 70 stations and black otherwise. You do not need to add any labels or popups to this map.

# Chloropleth Maps

A choropleth is a type of map that displays geographic units like countries, states, census areas, or watersheds with each unit colored based upon a particular variable. 

The first step in creating a choropleth is to define the boundaries of each geographic unit using spatial polygons.

In this example, we load the spatial polygons (an sp object) of country boundaries from a web url using the geojson_read function.

```R
shapeurl <- "https://raw.githubusercontent.com/kwaldenphd/leaflet-r/main/countries.geo.json"
WorldCountry <- geojson_read(shapeurl, what = "sp")

Map <- leaflet(WorldCountry) %>% addTiles() %>% addPolygons()
Map
```

Learning how to create your own spatial polygons is beyond the scope of this tutorial, but for most applications you can find a shapefile that someone else has created which contains the spatial polygon boundaries you need. 

Unfortunately I’m not aware of a single repository of different geo.json files, so finding the boundaries you need can sometimes require a bit of searching.

The addPolygons function will add the area boundaries for each geographic unit defined in sp file. To modify the color, or other attributes, of these polygons we can supply additional arguments:

```R
Map <- leaflet(WorldCountry) %>% addTiles() %>% 
  addPolygons(
  fillColor = "red",
  weight = 2,
  opacity = 1,
  color = "white",
  fillOpacity = 0.7)

Map
```

You should recognize that the arguments within addPolygons expects you to provide a vector of length 1 (like we did in the example above) or a vector that is the same length as the number of spatial polygons (what is needed to make a choropleth). 

In the example below, we merge data from the Happy Planet with the countries in our sp file, and then use the variable LifeExpectancy to create a choropleth:

```R
## Read in the Happy Planet data
happiness <- read.csv('https://raw.githubusercontent.com/kwaldenphd/leaflet-r/main/HappyPlanet.csv', stringsAsFactors = FALSE)

## Join the Happy Planet data to the countries in WorldCountry
CountryHappy <- left_join(data.frame(Name = WorldCountry$name), happiness, by = c("Name" ="Country"))

## Create a color palette (using the "magma" color scheme)
pal <- colorBin("magma", domain = CountryHappy$LifeExpectancy)


## Use the palette and LifeExpectancy to color the spatial polygons
Map <- leaflet(WorldCountry) %>% addTiles() %>% 
  addPolygons(
  fillColor = pal(CountryHappy$LifeExpectancy),
  weight = 2,
  opacity = 1,
  color = "white",
  fillOpacity = 0.7)
Map
```

A few things to note from this example.

We use left_join because every spatial polygon (country) needs to have a fillColor.

Not every spatial polygon defined in WorldCountry had a matching entry in the Happy Planet data, but addPolygons will use grey to color spatial polygons with an NA value for their fill color.

We must supply fill color via a color palette function, here we create this function using colorBin.

## Highlights

To add interactivity to our map we can use the highlight argument to indicate when the mouse is hovering over a particular spatial polygon.

```R
Map <- leaflet(WorldCountry) %>% addTiles() %>% 
  addPolygons(
    fillColor = pal(CountryHappy$LifeExpectancy),
    weight = 2,
    opacity = 1,
    color = "white",
    fillOpacity = 0.7,
    highlight = highlightOptions(
                   weight = 3,
                   color = "grey",
                   fillOpacity = 0.7,
                   bringToFront = TRUE))

Map
```

## Labels and Popups

In addition to adding highlights, we can add labels and popups to our spatial polygons.

```R
myLabels <- paste("<strong>", CountryHappy$Name, "</strong>", "<br/>", 
                   "Life Expectancy:", CountryHappy$LifeExpectancy)
myPopups <- paste("Happy Planet Rank", CountryHappy$HPIRank)

Map <- leaflet(WorldCountry) %>% addTiles() %>% 
  addPolygons(
    fillColor = pal(CountryHappy$LifeExpectancy),
    weight = 2,
    opacity = 1,
    color = "white",
    fillOpacity = 0.7,
    highlight = highlightOptions(
                   weight = 3,
                   color = "grey",
                   fillOpacity = 0.7,
                   bringToFront = TRUE),
    label = lapply(myLabels, HTML),
    popup = myPopups)

Map
```

A few things to note from this example.

The labels include HTML commands, but to get addPolygons to read the HTML commands properly we need to apply (using lapply) the function HTML from the htmltools package to each element of the vector we supply to the label argument.

If we aren’t using any HTML commands, like in our popups, we don’t need to worry about telling addPolygons that we’re using HTML.

## Legends

As a final step, we might consider adding a legend to our choropleth. The addLegend function will add a legend as a new layer:

```R
myLabels <- paste("<strong>", CountryHappy$Name, "</strong>", "<br/>", 
                   "Life Expectancy:", CountryHappy$LifeExpectancy)
myPopups <- paste("Happy Planet Rank", CountryHappy$HPIRank)

Map <- leaflet(WorldCountry) %>% addTiles() %>% 
  addPolygons(
    fillColor = pal(CountryHappy$LifeExpectancy),
    weight = 2,
    opacity = 1,
    color = "white",
    fillOpacity = 0.7,
    highlight = highlightOptions(weight = 3,
                   color = "grey",
                   fillOpacity = 0.7,
                   bringToFront = TRUE),
    label = lapply(myLabels, HTML),
    popup = myPopups) %>%
  addLegend(pal = pal, values = CountryHappy$LifeExpectancy,
            title = "Life Expectancy", position = "bottomright")

Map
```

# Practice Problems

Question #3:

The us_cities data set in the geojsonio package contains state capitals and all cities in the United States with populations larger than 40,000. 

The file located at “https://raw.githubusercontent.com/kwaldenphd/leaflet-r/main/us-states.json” contains spatial polygon boundaries for all 50 states (and the District of Columbia and Puerto Rico). 

The USArrests data set in the datasets package contains violant crime rates by state.

For this question you should create a map with the following features:

Spatial polygons for each state filled according to colors from the “viridis” color scheme and based upon that state’s murder rate in the USArrests data set.

Black outlines for each spatial polygon.

A legend in the bottom left that depicts the color scheme with an informative title.

A highlight that outlines a state in white when a user hovers over it.

Popups that show each state’s name in bold, along with its UrbanPop below the name on a new line.

Circle markers (with radius = 2) for each city in the us_cities data set where state capitals are colored “red” and all other cities are colored yellow

Labels for each city that display the city’s name (it is okay to keep the state abbreviation) and the city’s population

Hints: Use the argument bringToFront = FALSE in your highlight to avoid masking the city labels when hovering over a state

```R
shapeurl <- "https://raw.githubusercontent.com/kwaldenphd/leaflet-r/main/us-states.json"
UnitedStates <- geojson_read(shapeurl, what = "sp")
data("USArrests")
data("us_cities")
```

# Lab Notebook Questions
