---
title: Where to open a new Chipotle?
author: Tim Metzler
date: '2023-03-05'
slug: where-to-open-a-new-chipotle
categories: []
tags: ["Data Science"]
subtitle: ''
summary: 'A datacamp project using the leaflet package to find new possible Chiptole locations'
authors: []
lastmod: '2023-03-05T13:55:29+01:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: yes
projects: []
---

## 1. Chipotle locations from Thinknum
<p>Some of us have thought about opening a business and most of us have thought about grabbing a burrito from Chipotle. To do either of these things we need to know where the current Chipotle stores are (and are not) located. </p>
<p>In this notebook, the <code>leaflet</code> package in R and data from <a href="https://www.thinknum.com/">Thinknum</a> are used to find potential locations for new Chipotle restaurants as well as the closest place to grab a burrito. </p>
<p>Thinknum tracks thousands of websites capturing and indexing vast amounts of public data. Let's get started by loading and exploring data on every Chipotle restaurant tracked in the Thinknum data. Then we will build several <code>leaflet</code> maps that we can use to explore the data and to see where we might recommend opening a Chipotle.</p>




```r
library(leaflet)
library(leaflet.extras)
library(readr)
library(dplyr)
```

```
## 
## Attache Paket: 'dplyr'
```

```
## Die folgenden Objekte sind maskiert von 'package:stats':
## 
##     filter, lag
```

```
## Die folgenden Objekte sind maskiert von 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
# Read datasets/chipotle.csv into a tibble named chipotle using read_csv
chipotle <- read_csv("datasets/chipotle.csv")
```

```
## Rows: 2484 Columns: 8
```

```
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (4): street, city, st, ctry
## dbl (3): id, lat, lon
## lgl (1): closed
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```r
# Print out the chipotle tibble using the head function
head(chipotle)
```

```
## # A tibble: 6 × 8
##        id street                   city        st    ctry      lat    lon closed
##     <dbl> <chr>                    <chr>       <chr> <chr>   <dbl>  <dbl> <lgl> 
## 1 1358023 121 N. La Cienega Blvd., Los Angeles <NA>  United…  34.1 -118.  TRUE  
## 2 1358955 24369 Cedar Rd,          Lyndhurst   OH    United…  41.5  -81.5 TRUE  
## 3 1359012 1130 West Grove Ave.,    Mesa        <NA>  United…  33.4 -112.  TRUE  
## 4 1359490 6316 Delmar,             St. Louis   MO    United…  38.7  -90.3 TRUE  
## 5 1359574 1464 St. Louis Galleria, St. Louis   MO    United…  38.6  -90.3 TRUE  
## 6 1359575 8301 Westchester,        Dallas      TX    United…  32.9  -96.8 TRUE
```

## 2. Do Chipotle stores ever close?
<p>First, let's make sure we don't recommend opening a location where one has already been closed. This may also prevent many a disappointing Chipotle run to closed locations. Rather than looking just at the city/state pairs in the data, we can plot all of the closed locations to see exactly where the restaurants were located. </p>
<p><code>leaflet</code> maps work with the <code>%&gt;%</code> operator so we can pipe our <code>chipotle</code> data directly into a chain of function calls to produce an interactive map. All of the Chipotle locations have already been geocoded. The <code>leaflet</code> package will scan our column names for variables that are likely lat and lon and if we use a common naming convention (e.g., lat/lon, latitude/longitude, or lat/lng), <code>leaflet</code> will automatically know which columns contain our coordinates. </p>
<p>Because of their interactive features <code>leaflet</code> maps can be especially helpful for exploratory data analyses. After we make our first map take a minute or two to zoom and pan the map to explore where Chipotle locations have closed. </p>


```r
# Create a leaflet map of all closed Chipotle stores
closed_chipotles <- 
chipotle %>% 
  # Filter the chipotle tibble to stores with a value of t for closed
  filter(closed == TRUE) %>% 
  leaflet() %>% 
  # Use addTiles to plot the closed stores on the default Open Street Map tile
  addTiles() %>%
  # Plot the closed stores using addCircles
  addCircles() 
```

```
## Assuming "lon" and "lat" are longitude and latitude, respectively
```

```r
# Print map of closed chipotles
closed_chipotles
```



<iframe seamless = "" width = "100%" height = "500" class="shortcode-iframe" src="/leaflet/closed_chipotles.html"></iframe>

## 3. How many Chipotle stores have closed?
<p>After exploring the map, the first question that comes to mind is why did these particular locations close? In fact, why would any Chipotle ever close? Unfortunately, questions like this defy logic, so after quickly counting up the closed locations, this notebook moves on to the more important question of "Where should the next Chipotle be opened?". </p>
<p>Rather than counting up all of the circles on our interactive map, we can use <code>dplyr</code> to quickly count the number of closed stores. After we note this, we'll create a new <code>tibble</code> that removes the closed locations from our data so that we do not confuse them for open locations in future maps. </p>



```r
# Use count from dplyr to count the values for the closed variable
chipotle %>% 
  count(closed)
```

```
## # A tibble: 2 × 2
##   closed     n
##   <lgl>  <int>
## 1 FALSE   2469
## 2 TRUE      15
```

```r
# Create a new tibble named chipotle_open that contains only open chipotle 
chipotle_open <-
  chipotle %>% 
  filter(closed == FALSE) %>% 
  # Drop the closed column from chipotle_open
  select(-closed)
```


## 4. Where are (and aren't) there Chipotles?
<p>Where's the closest Chipotle? Perhaps, more interesting is a slightly different question, where aren't there Chipotles (in the US)? 
By mapping all of the Chipotle locations on an interactive <code>leaflet</code> map we can start to explore patterns in the geographic distribution of the chain's locations. </p>
<p>Since there are thousands of store locations, many of which are clustered closely together, we will start with a heatmap. Heatmaps are a popular option for mapping large amounts of points because they leverage a color scheme to represent the data rather than plotting each point individually. This enables users to quickly identify variation in the density of points and prevents tightly clustered points from overlapping. </p>
<p>The <code>addHeatmap</code> function comes from the <code>leaflet.extras</code> package, which contains many useful functions that extend the <code>leaflet</code> package.</p>
<p>Zooming and panning the map the heatmap will adjust based on the current view of the map. </p>
<p>Are there any Chipotle deserts in the United States? </p>




```r
# Pipe chipotle_open into a chain of leaflet functions
chipotle_heatmap <- 
chipotle_open %>% 
  leaflet() %>% 
  # Use addProviderTiles to add the CartoDB provider tile 
  addProviderTiles("CartoDB") %>%
  # Use addHeatmap with a radius of 8
  addHeatmap(radius = 8)
```

```
## Assuming "lon" and "lat" are longitude and latitude, respectively
```

```r
# Print heatmap
chipotle_heatmap 
```



<iframe seamless = "" width = "100%" height = "500" class="shortcode-iframe" src="/leaflet/chipotle_heatmap.html"></iframe>


## 5. Which States have the fewest Chipotles?
<p>Using the greyscale <code>CartoDB</code> provider tile with a colorful heatmap palette quickly revealed both the presence and absence of Chipotle stores throughout the United States. Using a greyscale base map is often useful for exploratory data analysis as it  makes patterns of Chipotle clusters and Chipotle deserts clearly stand out on the map. </p>
<p>For example, panning and zooming the map reveals that Chipotles are often located on horizontal or vertical lines. Zooming in further reveals that this is because stores are often located near interstate highways (check out Utah for an example). </p>
<p>Let's take a closer look at where there are not Chipotle stores by quantifying the Chipotle deserts using <code>dplyr</code> to count the number of Chipotle locations in each US state. </p>



```r
# Create a new tibble called chipotles_by_state to store the results
chipotles_by_state <- 
chipotle_open %>% 
  # Filter the data to only Chipotles in the United States
  filter(ctry == "United States") %>% 
  # Count the number of stores in chipotle_open by st
  count(st) %>% 
  # Arrange the number of stores by state in ascending order
  arrange(n)

# Print the state counts
chipotles_by_state
```

```
## # A tibble: 48 × 2
##    st        n
##    <chr> <int>
##  1 MS        1
##  2 ND        1
##  3 VT        1
##  4 WY        2
##  5 MT        3
##  6 ME        5
##  7 WV        5
##  8 AR        6
##  9 DE        6
## 10 NH        7
## # ℹ 38 more rows
```

## 6. How many States in the US?
<p>The <code>chipotle_by_state</code> tibble had 48 rows, but there are 50 fifty states in the US. Why don't we have fifty rows? Perhaps, there are two (unfortunate) states do not have a single Chipotle. Let's take a look using a couple of handy features that are included in base R. The <code>state.abb</code> vector has fifty elements, each containing a state's abbreviation. Using this vector in combination with the <code>!</code> and <code>%in%</code> operators, we can quickly and systematically determine which states do not have a Chipotle. The <code>%in%</code> operator helps us to determine which elements of one vector are present <em>in</em> another vector, while the <code>!</code> operator allows us to accomplish the inverse (i.e., finding which elements of one vector are <em>not in</em> another). </p>




```r
# Print the state.abb vector
state.abb
```

```
##  [1] "AL" "AK" "AZ" "AR" "CA" "CO" "CT" "DE" "FL" "GA" "HI" "ID" "IL" "IN" "IA"
## [16] "KS" "KY" "LA" "ME" "MD" "MA" "MI" "MN" "MS" "MO" "MT" "NE" "NV" "NH" "NJ"
## [31] "NM" "NY" "NC" "ND" "OH" "OK" "OR" "PA" "RI" "SC" "SD" "TN" "TX" "UT" "VT"
## [46] "VA" "WA" "WV" "WI" "WY"
```

```r
# Use the %in% operator to determine which states are in chipotles_by_state
state.abb %in% chipotles_by_state$st
```

```
##  [1]  TRUE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE FALSE  TRUE
## [13]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
## [25]  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
## [37]  TRUE  TRUE  TRUE  TRUE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE  TRUE
## [49]  TRUE  TRUE
```

```r
# Use the %in% and ! operators to determine which states are not in chipotles_by_state
!state.abb %in% chipotles_by_state$st
```

```
##  [1] FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE FALSE
## [13] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [25] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [37] FALSE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
## [49] FALSE FALSE
```

```r
# Create a states_wo_chipotles vector
states_wo_chipotles <- state.abb[!state.abb %in% chipotles_by_state$st]

# Print states with no Chipotles
states_wo_chipotles
```

```
## [1] "AK" "HI" "SD"
```

## 7. Where to open a Chipotle I
<p>Wait a second…have I had this wrong all along? 48 + 3 = 51 states!?! Let's take a closer look at the values in our state variable that are not in the <code>state.abb</code> vector. </p>
<pre><code class="{r} language-{r}">chipotles_by_state$st[!chipotles_by_state$st %in% state.abb]
</code></pre>
<p>Ah! Washington, D.C. surprises me once again. DC is a district and not a state, so 51 seems ok for this notebook. </p>
<p>Let's focus on the only state in the contiguous United States that does not have a Chipotle: South Dakota. If we were to open a Chipotle location in South Dakota, how might we go about selecting proposed locations? In the following chunks of code, we look at several maps to explore how the location of current Chipotles as well as geographic, transportation, and governmental features of the state may inform this decision.  </p>
<p>First, let's take a look at how South Dakota's population is distributed across the state using data from the US Census. The <code>tidycensus</code> package in R can be used to facilitate access to census data from R. For a great example, check out [this](
https://juliasilge.com/blog/using-tidycensus/) blog post by Julia Silge.  </p>




```
## old-style crs object detected; please recreate object with a recent sf::st_crs()
## old-style crs object detected; please recreate object with a recent sf::st_crs()
## old-style crs object detected; please recreate object with a recent sf::st_crs()
## old-style crs object detected; please recreate object with a recent sf::st_crs()
```




<iframe seamless = "" width = "100%" height = "500" class="shortcode-iframe" src="/leaflet/sd_pop_map.html"></iframe>

## 8. Where to open a Chipotle II
<p>Minnehaha and Pennington counties really stand out on population map. These counties are home to Sioux Falls and Rapid City, respectively. Let's take a closer look at each of the two largest cities in South Dakota to consider what features of the base map may be important when selecting the location of a Chipotle?</p>
<p>Sioux Falls has a larger population, but Rapid City is proximate to Badlands National park, which has a million visitors a year. Additionally, we should note that I-90, a major interstate in America, runs through both cities.  </p>
<p>Let's plot a proposed Chipotle location in each city to further our exploration. The <code>geocode</code> function from the <code>ggmap</code> package can be used to geocode locations in R. Alternatively, the <code>addReverseSearchOSM</code> function from the <code>leaflet.extras</code> package can be used to enable geocoding via mouse click on an interactive map. </p>


```r
# Load chipotle_sd_locations.csv that contains proposed South Dakota locations  
chipotle_sd_locations <- read_csv("datasets/chipotle_sd_locations.csv")
```

```
## Rows: 2 Columns: 5
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (3): city, st, status
## dbl (2): lat, lon
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

```r
# limit chipotle store data to locations in states boardering South Dakota
chipotle_market_research <- 
  chipotle_open %>% 
  filter(st %in% c("IA", "MN", "MT", "ND", "NE", "WY")) %>% 
  select(city, st, lat, lon) %>% 
  mutate(status = "open")# %>% 
  # bind the data on proposed SD locations onto the open store data
 # bind_rows(chipotle_sd_locations) 

# print the market research data
chipotle_market_research
```

```
## # A tibble: 90 × 5
##    city             st      lat   lon status
##    <chr>            <chr> <dbl> <dbl> <chr> 
##  1 Lincoln          NE     40.8 -96.6 open  
##  2 Brooklyn Park    MN     45.1 -93.4 open  
##  3 Eagan            MN     44.8 -93.2 open  
##  4 Champlin         MN     45.2 -93.4 open  
##  5 Woodbury         MN     44.9 -92.9 open  
##  6 Columbia Heights MN     45.1 -93.2 open  
##  7 Fargo            ND     46.9 -96.9 open  
##  8 Iowa City        IA     41.7 -91.5 open  
##  9 Minnetonka       MN     44.9 -93.5 open  
## 10 Minneapolis      MN     45.0 -93.3 open  
## # ℹ 80 more rows
```

## 9. Where to open a Chipotle III
<p>Let's map our proposed Chipotle restaurants in Sioux Falls and Rapid City so we can quickly see how close they are to the nearest open location. </p>
<p>So far our maps have been built using a single base map (e.g., <code>CartoDB</code>) with a single data layer (e.g., circle markers or polygons).  The <code>leaflet</code> package works like <code>ggplot2</code> and allows for multiple data layers on the same map.  Then <code>addLayerControls</code> will enable users to have control of which layers are visible. </p>
<p>Let's apply this concept in a new <code>leaflet</code> map that plots all of the open and proposed Chipotle locations in South Dakota and its bordering states. Then adding a second layer to draw a circle around each of the proposed locations to determine if there is an open store within 100 miles. </p>
<p>When using a categorical variable to create a color palette, colors can be mapped directly to the levels of the factor (i.e., there is one color in the palette for each level of the factor) or the palette can be interpolated by the <code>colorFactor</code> function to create the necessary number of colors. </p>



```r
# Create a blue and red color palette to distinguish between open and proposed stores
pal <- colorFactor(palette = c("Blue", "Red"), domain = c("open", "proposed"))

# Map the open and proposed locations
sd_proposed_map <-
  chipotle_market_research %>% 
  leaflet() %>% 
 #Add the Stamen Toner provider tile
  addProviderTiles("Stamen.Toner") %>%
# Apply the pal color palette
  addCircles(color = ~pal(status)) %>%
  # Draw a circle with a 100 mi radius around the proposed locations
  addCircles(data = chipotle_sd_locations, radius = 100 * 1609.34, color = ~pal(status), fill = FALSE)  
```

```
## Assuming "lon" and "lat" are longitude and latitude, respectively
## Assuming "lon" and "lat" are longitude and latitude, respectively
```

```r
# Print the map of proposed locations 
sd_proposed_map
```



<iframe seamless = "" width = "100%" height = "500" class="shortcode-iframe" src="/leaflet/sd_proposed_map.html"></iframe>

## 10. Where to open a Chipotle IV
<p>It looks like there is a Chipotle within a 100 miles of the proposed Sioux Falls location, but not Rapid City. This is helpful to know but perhaps even more helpful would be to understand all of the locations that are closer to a proposed Chipotle than to an open one. </p>
<p>Voronoi polygons can be used to plot a polygon around each location. The bounds of each polygon will enclose all of the points on the map that are closest to a specific Chipotle. These polygons can then be used to visualize an approximation of the area covered by each Chipotle.  </p>
<p>Where should the next Chipotle be opened (hint: there has never been a wrong answer to this question!)? </p>



```r
# load the Voronoi polygon data 
load("datasets/polys.Rda")

voronoi_map <- 
  polys %>%
  leaflet() %>%
  # Use the CartoDB provider tile
  addProviderTiles("CartoDB") %>%
  # Plot Voronoi polygons using addPolygons
  addPolygons(fillColor = ~pal(status), weight = 0.5, color = "black") %>%
  # Add proposed and open locations as another layer
  addCircleMarkers(data = chipotle_market_research, label = ~city, color = ~pal(status))
```

```
## Assuming "lon" and "lat" are longitude and latitude, respectively
```

```
## old-style crs object detected; please recreate object with a recent sf::st_crs()
## old-style crs object detected; please recreate object with a recent sf::st_crs()
## old-style crs object detected; please recreate object with a recent sf::st_crs()
## old-style crs object detected; please recreate object with a recent sf::st_crs()
```

```r
# Print the Voronoi map
voronoi_map
```



<iframe seamless = "" width = "100%" height = "500" class="shortcode-iframe" src="/leaflet/voronoi_map.html"></iframe>

## 11. Where should the next Chipotle be opened?
<p>After loading thousands of Chipotle locations from Thinknum, creating tables and a heatmap, identifying the shocking fact that South Dakota does not have a single Chipotle, using exploratory data analysis to investigate where a store location may make sense, including</p>
<ul>
<li>creating a county-level choropleth map showing population,</li>
<li>drawing circles with 100-mile radii around each proposed location, and</li>
<li>mapping Voronoi polygons to estimate the area covered by each proposed Chipotle, </li>
</ul>
<p>it seems that Rapid City, SD would be the best choice in regard to the area covered by the new location (zoom in if you can't see the cities). 
