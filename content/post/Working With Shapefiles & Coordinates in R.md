
+++
title = "Working With Shapefiles and Coordinates in R"
author = "Jason Geslois, MAS, MPH, MCHES"
date = "2019-09-02"
+++


## Mapping Counties

R is a great program to do a variety of data science projects, solve statistical problems, as well as being a solid way to map GIS projects. It uses literally thousands of packages that enhance and improve its capabilities of what you can do. In public health, there are many times you may need a map, but one of the simplest ways to do it is in using and learning different packages. For example to get a map of Texas counties, you could do the following:

```{r}
library(maps)
map('county', 'texas', fill = TRUE, col = palette())
```

{{< figure library="1" src="palettetexas.jpg" title="" lightbox="true" >}}


This looks pretty good and is a very simple way to quickly get a map made. Another way you may need to see it would be to look at it interactively and use something like a leaflet map. To get that, let's import some data and make some shapefiles to do that with. 


## Working With Shapefiles

What is nice about R is that you can download and make just about any type of shapefile you may want as a backdrop or to use in any projects you may have to work on. The Database of Global Administrative Areas (GADM) provides shapefiles for the whole world. Below is how to download the data, and then once you have the data, to save it for future use. If you have trouble, you can visit their site here and download directly: https://gadm.org/data.html. If you use the code below remember to remove the `#` symbol in front of either line. 


```{r}
library(raster)
library(sp)
#Use this line to download the County Shapefile for the US
#us<-getData('GADM', country='USA', level=2)  

#If you save the file, to do again in the future, you can use this line to load it without downloading again 
 us <- readRDS("gadm36_USA_2_sp.rds") 
```

You can do this for most of the world, but in this example we're using the US. And being that I am in Texas lets partition it down to that level, and then once more to a selection of counties. In this example, I'll use the counties that I work in at my health district. 

```{r}
texas <- subset(us,NAME_1=="Texas")
plot(texas)
```
{{< figure library="1" src="countytexas.jpg" title="" lightbox="true" >}}


Now for the district map. 

```{r}
mapdist <- subset(texas, NAME_2=="Smith" | NAME_2=="Anderson" | NAME_2=="Wood" | NAME_2=="Van Zandt" |NAME_2=="Rains" | NAME_2=="Gregg" | NAME_2=="Henderson")
plot(mapdist, col='pink')
```
{{< figure library="1" src="pinkdistrict.jpg" title="" lightbox="true" >}}


If you want to see them together on the same map, we can use the package `ggplot2` inside the `tidyverse` package to make a nice view of where these counties are in Texas. 

```{r}
library(tidyverse)
states <- map_data("state")
tx_county <- subset(states, region == "texas")
tx_base <- ggplot(data = tx_county, mapping = aes(x = long, y = lat, group = group)) + 
  coord_fixed(1.3) + 
  geom_polygon(color = "black", fill = NA)
tx_base + theme_void() +
  geom_polygon(data = texas, fill = NA, color = NA) +
  geom_polygon(data = mapdist, color = "black", fill = "pink")  # this puts the state border back on top
```
{{< figure library="1" src="txdistrict.jpg" title="" lightbox="true" >}}

Now doing this, let's use the `Leaflet` package to make an interactive map. In this package, it is presented very aesthetically. 

```{r}
library(leaflet)
leaflet(mapdist) %>%
    addPolygons()%>%
    addTiles() 
```



If you wanted to save either of these areas or whatever selection you're working with, you could do so by saving them as a shapefile to export out to use in another tool like ArcGIS using the below code. This will save the files in an temporary directory in your current working directory. I am using the `overwrite_layer = TRUE` command here to test it as I save it, you can remove it if you need. 

```{r}
library(rgdal)
#outputs your shape polygons or spatial point data into a temp folder with all shape files for the state of Texas
writeOGR(obj=texas, dsn="tempdir", layer="texas", driver="ESRI Shapefile", overwrite_layer = TRUE)  

#just the counties of your selection, in my case "the district" that we created above
writeOGR(obj=mapdist, dsn="tempdir", layer="mapdist", driver="ESRI Shapefile", overwrite_layer = TRUE) 
```

For future use, if you want to read the shapefiles back into R, you can use the following code: 

```{r}
library(rgdal)
s <- readOGR(dsn=path.expand("mapdist.shp"), layer="mapdist")
```

## Plotting Random Points

So now you've got your shapefile and you want to plot some random points in your area. This could represent controls you use in a case control study, site locations you want to sample, or a variety of other uses etc. 

```{r}
library(raster)
library(GISTools)
library(rgdal)
library(sp)

#n is the number of points to create
plot(s) ; points(spsample(s, n=100, type='random'), col='red', pch=3, cex=0.5)
```

{{< figure library="1" src="pts1.jpg" title="" lightbox="true" >}}

These points could be reverse geocoded to get their exact coordinates as well as their approximate nearest address if we wanted to go that route. There's a few ways to do that, however I want to skip this step and instead produce some random coordinates within this area that we could use. 

So to do that, we need to start by figuring out what our boundaries are of the shapefile within our district area. So we will need to look at it using the `bbox()` function and assign it to a variable for use later.  

```{r}
bbox(mapdist)
bbs <- bbox(mapdist)
```

Now that we know our boundaries of longitude and latitudes, we want to create random samples of lat/long points within this box. We need to setup how to use the Min X, Min Y, Max X, and Max Y values so that whatever coordinates you may have, can be input into the random generator. In each of those values of the bbox, they represent a element as [1], [2], [3], and [4]. So let's assign those values to the mins and maxs. 

```{r}
#assigns the min and max boundaries of your reference bbox to create your samples from within it
minx <- bbs[1]
minx
miny <- bbs[2]
miny
maxx <- bbs[3]
maxx
maxy <- bbs[4]
maxy
```

Now let's put it in the charlatan package to randomly generate 100 lat/long points. You'll want to `View()` them so you can see exactly how they are structured. 
```{r}
library(charlatan)
set.seed(23)
pts100 <- data.frame(ch_position(n = 100, bbox = c(minx, miny, maxx, maxy)))
head(pts100)
```

Well that's not good. When we run this, it turned out that they are stretched out in rows and that just isn't helpful to what we need. Let's fix this to make 2 nice columns. 

```{r}
#transposes the data, basically switching the rows and columns; the t() transposes, while the as.data.frame turns it into a data frame to override an error
pts100fix <- as.data.frame(t(pts100))
head(pts100fix)
```

Much better. But let's name the columns so it looks better and get rid of the first garbled column. 

```{r}
clean <- as_tibble(pts100fix)
colnames(clean) <- c("long", "lat")  #gives the names to the columns
head(clean)
```

Now to plot and test our results. 


```{r}
plot(s) ; points(clean, col='red', pch=3, cex=0.5)
```
{{< figure library="1" src="pts2.jpg" title="" lightbox="true" >}}

Something didn't work right. Looks like our boundary box encompasses the whole region inside and outside. So to fix this we have to convert our regular data frame with our lat/long points to a spatial data frame object with projection. Further explanation and help can be found here. 

conversion help: 
https://stackoverflow.com/questions/29736577/how-to-convert-data-frame-to-spatial-coordinates 


```{r}
xy <- clean[,c(1,2)]

spdf <- SpatialPointsDataFrame(coords = xy, data = clean,
                               proj4string = CRS("+proj=longlat +datum=WGS84 +ellps=WGS84 +towgs84=0,0,0"))

```


Now that it's in the appropriate spatial data object, we need to transform the points to those of our county shapefile so we can clip around that area. 

clipping help from here: 
https://www.r-bloggers.com/clipping-spatial-data-in-r/ 


```{r}
library(rgdal)
clean <- spTransform(spdf, CRS(proj4string(s))) # transform CRS
clean_subset <- clean[s, ]
plot(s); points(clean_subset, col='red', pch=3, cex=0.5)
```

{{< figure library="1" src="pts3.jpg" title="" lightbox="true" >}}

That did the trick. The last bit on this post will be a little bit of spatial statistics to look at the spatial intensity of the coordinates we have now. 

## Spatial Intensity

Making a map of spatial intensity can help us look at how concentrated each of the points are in each area. one way to examine this is using a kernel density method. This method uses a smoothing function to illustrate the spatial variation in each of the point data. This can be useful to assess clusters, and look at trend patterns in data that is spatially distributed. 

More information can be found here: 
https://mgimond.github.io/Spatial/point-pattern-analysis-in-r.html 

Let's load our packages well need. 
```{r}
library(spatstat)
library(splancs)
library(maptools)
library(maps)
library(rgdal)
```

The spatstat package is used here to do the process. There is quite a lot of features in this package so definitely read the vignette if you get lost. First we need to convert our point data to a ppp object which is how the packages handles them. 

```{r}
pppclean <- as.ppp(clean_subset)
```

Now we need to convert our shapefile to an owin object, another file type it uses.

```{r}
sowin <- as(s, "owin") 
```

Finally we bind the owin object to our ppp object in what's called a Window object so that it maintains our area of interest and will be defined. 

```{r}
Window(pppclean) <- sowin
```

Now we'll run the kernel density function. Each point is represented in an area of square meters. You can adjust the bandwidth for various purposes with the `sigma=` variable. Using the `edge=` corrects for edge affects that sometimes occur around the border areas. 

```{r}
K2 <- density(pppclean, edge=TRUE, sigma=0.07) # Using a low bandwidth
plot(K2, main='Kernel Density Plot', las=1)
contour(K2, add=TRUE)
```

{{< figure library="1" src="kerneldistrict.jpg" title="" lightbox="true" >}}

There is a lot of math and much more detailed aspects to how one of these is created, but I just wanted to show how this can be just one useful tool in public health. This wraps up this particular post. 



