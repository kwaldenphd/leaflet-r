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

MORE ON LEAFLET

# Data and Environment Setup

## Packages

We'll be using a number of packages in this lab.
- leaflet
- geojsonio
- dplyr
- htmltools



```R
# install.packages("leaflet")
# install.packages("geojsonio")
# install.packages("dplyr")
# install.packages("htmltools")

library(leaflet)    # The map-making package
library(geojsonio)  # A package for geographic and spatial data, requires the latest version of dplyr
library(dplyr)      # Used for data manipulation and merging
library(htmltools)  # Used for constructing map labels using HTML
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
happiness <- read.csv('raw.githubusercontent.com/kwaldenphd/leaflet-r/main/HappyPlanet.csv', stringsAsFactors = FALSE)
head(happiness)
```

# Base Maps

# Maps with Markers

## Marker Clusters

## Circle Markers

## Custom Markers

# Chloropleth Maps

## Highlights

## Labels and Popups

## Legends

# Practice Problems

# Additional Resources

# Lab Notebook Questions
