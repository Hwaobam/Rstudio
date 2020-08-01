---
title: "Everest 3D Map Generation"
output:
  html_notebook: default
  html_document:
    df_print: paged
---


Various number of packages which are required for the 3D map generation. These are installed and loaded into the program. These packages avail the functions such as Plotting of points, shading and adjusting the colors, conversion of GIS data, validating GDAL operations,etc. 
```{r}
library(rayshader)
library(magick)
library(sp)
library(raster)
library(scales)
library(rgdal)
library(rgeos)
library(av)
```
The GIS data for elevation is downloaded from Derek Watkin's -"30 meter SRTM tile downloader". The downloaded SRTM hgt data is converted to a matrix. The result of the plot is an elevation intensity profile of the tile.
```{r fig1, fig.height = 15, fig.width = 10, align= "center"}
elevation1 = raster::raster("D:/Assam maps/New Folder/everest/N27E086.hgt")
elevation2 = raster::raster("D:/Assam maps/New Folder/everest/N27E087.hgt")
everest_elevation = raster::merge(elevation1,elevation2)
height_shade(raster_to_matrix(everest_elevation)) %>%
plot_map()
```
The coordinate reference of the elevation data matched to the imagery data
```{r}
crs(everest_elevation)
```

The desired area whose map is to be plot is cropped in a rectangular shape by defining the co-ordinates of the bottom left and top right, diagonally opposite points. The longitude and latitude of the two points are found using "Google Maps" and then defined.
```{r}
bot_lefty=86.856740
bot_leftx=27.69857
top_righty=87.308946
top_rightx=27.991321
distl2l= top_righty-bot_lefty  
scl=1-(20/(distl2l*111))
bottom_left = c(bot_lefty, bot_leftx)
top_right   = c(top_righty, top_rightx)
extent_latlong = SpatialPoints(rbind(bottom_left, top_right), proj4string=sp::CRS("+proj=longlat +ellps=WGS84 +datum=WGS84"))
e = extent(extent_latlong)
e
scl
```
Then, the elevation data is converted into Base R matrix
```{r fig4, fig.height = 15, fig.width = 10, align= "center"}
elevation_cropped = crop(everest_elevation, e)
everestel_matrix = raster_to_matrix(elevation_cropped)
```

The geographical plot with 3d visualization
```{r fig7, fig.height = 12, fig.width = 9, align= "center"}
everestel_matrix %>%
 
  sphere_shade(texture = "bw") %>%
  plot_3d( everestel_matrix, windowsize = c(1100,900), zscale = 15, shadowdepth = -15,
           zoom=0.5, phi=45,theta=-45,fov=70, background = "#F2E1D0", shadowcolor = "#523E2B")
render_scalebar(limits=c(0,10,20),label_unit = "km",position = "S", y=50,scale_length = c(scl,1))
render_compass(position = "W" )
render_snapshot(title_text = "Mount Everest_geographical|Imagery: Landsat 8 | DEM: 30m SRTM",title_bar_color = "#1f5214", title_color = "white", title_bar_alpha = 1)
```
optional: Conversion of the map to a video footage of its planar rotation
```{r}
angles= seq(0,360,length.out = 1441)[-1]
for(i in 1:1440) {
render_camera(theta=-45+angles[i])
  render_snapshot(filename = sprintf("evr_geo%i.png", i), 
                  title_text = "Everest_geographical| DEM: 30m SRTM",
                  title_bar_color = "#1f5214", title_color = "white", title_bar_alpha = 1)
}
rgl::rgl.close()
system("ffmpeg -framerate 60 -i evr_geo%d.png -vcodec libx264 -an Mount_Everest_geographical_1.mp4 ")
```
![Himalayas_geographical](C:/Users/asus/Documents/Mount_Everest_geographical_1.mp4)
