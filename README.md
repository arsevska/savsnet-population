# savsnet-population
This is a representation of the SAVSNET cat and dog population in Great Britain
# Count the SAVSNET cat and dog population!

This code creates a shape file with the distribution and number of cats, dogs and all together per 5km2 gridded cell of Great Britain (GB).

### Load libraries
library(rgdal)
library(raster)
library(sp)
library(maptools) # read/ analyse spatial data
library(plyr) # for counting, manipulating data frames

### Define EPSG string for GB
GBgrid = "+init=epsg:27700"

### Read the GB 5km grided cell and project to GB EPSG

Point to a location where you keep the shape file of the 5km2 gridded cells of GB. This dataset can be downloaded from here: https://digimap.edina.ac.uk/

GBGrid5km <- readOGR("C:/Users/arsevska/Dropbox/Ticks/Ticks_spatial_short/OSGB", "OSGB_Grid_5km_clipped_nuts0") # read the OSGB grid 
#plot(GBGrid5km)
GBGrid5km<-spTransform(GBGrid5km, CRS(GBgrid)) # transform the projection into GB grid
proj4string(GBGrid5km) # transform to GB national grid
colnames(GBGrid5km@data)<-tolower(colnames(GBGrid5km@data)) # get all columns to lower case

### Get the unique data on cats and dogs 

cats<- read.csv("M:/cats_unique_animals.csv")
dogs<-read.csv("M:/dogs_unique_animals.csv")

#### Remove the NA data 
cats<-cats[complete.cases(cats), ] # 655.930 cats
dogs<-dogs[complete.cases(dogs), ] # 1.580.796 dogs

### Transformation of the tick case data into spatial points ##### 

#### Create coordinates variable for cats 
coord_cats <- cbind(cats$oseast1m,cats$osnrth1m)
#### Create the SpatialPointsDataFrame for the ticks 
cats_sp <- SpatialPointsDataFrame(coord_cats, 
                                  data = cats, 
                                  proj4string = CRS(GBgrid))
head(cats_sp@coords)
projection(cats_sp)

#### Create coordinates variable for dogs
coord_dogs <- cbind(dogs$oseast1m,dogs$osnrth1m)
#### Create the SpatialPointsDataFrame
dogs_sp <- SpatialPointsDataFrame(coord_dogs, 
                                  data = dogs, 
                                  proj4string = CRS(GBgrid))
head(dogs_sp@coords)
projection(dogs_sp)

### Count the number of tick consults per 5km2 gridded cell

CATS
cats_5km<-over(cats_sp, GBGrid5km)
cats_5km_freq<-count(cats_5km, "tile_name")
names(cats_5km_freq)[names(cats_5km_freq)=="freq"] <- "freq_cats"
#cats_5km_freq$freq_cats<-as.numeric(cats_5km_freq$freq_cats)

DOGS
dogs_5km<-over(dogs_sp, GBGrid5km)
dogs_5km_freq<-count(dogs_5km, "tile_name")
names(dogs_5km_freq)[names(dogs_5km_freq)=="freq"] <- "freq_dogs"
#dogs_5km_freq$freq_dogs<-as.numeric(dogs_5km_freq$freq_dogs)

### Merge the frequency tables with the GBGrid 5km cells 

CATS
GBGrid5km<-merge(GBGrid5km, cats_5km_freq, by="tile_name")
summary(GBGrid5km$freq_cats) 

#10401 cells and 6507 NA values = 6507/10401 = 62.56%
#Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
#1.0     5.0    21.0   168.1    94.0  7495.0    6507 

DOGS
GBGrid5km<-merge(GBGrid5km, dogs_5km_freq, by="tile_name")
summary(GBGrid5km$freq_dogs) 

#10401 cells and 5058 NA values = 5058/10401 = 48.62%
#Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
#1.0     6.0    29.0   295.2   165.0 17210.0    5058 

ALL (Sum cats and dogs)
GBGrid5km$freq_pets<- GBGrid5km$freq_cats + GBGrid5km$freq_dogs
summary(GBGrid5km$freq_pets) 

#10401 cells and 6611 NA values = 6611/10401 = 63.56%
#Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
#2.0    29.0   104.0   586.1   407.0 24700.0    6611 

#### Save the file as a shape file
writeOGR(GBGrid5km, dsn = 'M:/Savsnet_cat_dog_population', 
         layer = 'Savsnet_cat_dog_population_2017', 
         driver = "ESRI Shapefile") # save the shp file

It's very easy to make link to external files Just [click the following link to see how !]( https://datanywhere.liv.ac.uk/?linkid=KZi4zr6VWWXG1EL8SwljnbUDrAz2tqxO3IDODS6ncAGbVD1e5WaElg)


```r
    writeOGR(GBGrid5km, dsn = 'M:/Savsnet_cat_dog_population', 
         layer = 'Savsnet_cat_dog_population_2017', 
         driver = "ESRI Shapefile")
```
