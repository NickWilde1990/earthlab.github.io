---
layout: single
title: "Crop a spatial raster dataset using a shapefile in Python"
excerpt: "This lesson covers how to crop a raster dataset and export it as a
new raster in Python"
authors: ['Leah Wasser']
modified: 2018-08-29
category: [courses]
class-lesson: ['intro-lidar-raster-python']
permalink: /courses/earth-analytics-python/raster-lidar-intro/crop-raster-data-in-python/
nav-title: 'Crop a raster'
week: 3
course: "earth-analytics-python"
sidebar:
  nav:
author_profile: false
comments: true
order: 7
topics:
  reproducible-science-and-programming: ['python']
  remote-sensing: ['lidar']
  earth-science: ['vegetation']
  spatial-data-and-gis: ['raster-data']
---

{% include toc title="In This Lesson" icon="file-text" %}

<div class='notice--success' markdown="1">

## <i class="fa fa-graduation-cap" aria-hidden="true"></i> Learning Objectives

After completing this tutorial, you will be able to:

* Crop a raster dataset in `Python` using a vector extent object derived from a shapefile.
* Open a shapefile in `Python`.

## <i class="fa fa-check-square-o fa-2" aria-hidden="true"></i> What You Need

You will need a computer with internet access to complete this lesson.

{% include/data_subsets/course_earth_analytics/_data-colorado-flood.md %}

</div>

In this lesson, you will learn how to crop a raster dataset in `Python`. Previously,
you reclassified a raster in `Python`, however the edges of your raster dataset were uneven.
In this lesson, you will learn how to crop a raster - to create a new raster
object / file that you can share with colleagues and / or open in other tools such
as QGIS.

## About Spatial Crop

Cropping (sometimes also referred to as clipping), is when you subset or make a dataset smaller, 
by removing all data outside of the crop area or spatial extent. In this case you have a large 
raster - but let's pretend that you only need to work with a smaller subset of the raster. 

You can use the `crop_extent` function to remove all of the data outside of your study area.
This is useful as it:

1. Makes the data smaller and 
2. Makes processing and plotting faster

In general when you can, it's often a good idea to crop your raster data!

To begin let's load the libraries that you will need in this lesson. 


## Load Libraries

### Be sure to set your working directory
`os.chdir("path-to-you-dir-here/earth-analytics/data")`

{:.input}
```python
import rasterio as rio
from rasterio.plot import show
import numpy as np
import os
import matplotlib.pyplot as plt
import geopandas as gpd
import earthpy as et
import cartopy as cp
plt.ion()
```

## Open Raster and Vector Layers

Next, you will use `rio.open()` to open a raster layer. Open and plot the canopy height model (CHM) that you created in the previous lesson.

{:.input}
```python
with rio.open("data/colorado-flood/spatial/boulder-leehill-rd/outputs/lidar_chm.tif") as lidar_chm:
    lidar_chm_im = lidar_chm.read(masked = True)[0]


fig, ax = plt.subplots()
show(lidar_chm_im, cmap='terrain', ax=ax)
ax.set_title("Lidar Canopy Height Model (CHM)", fontsize = 16);
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}//images/courses/earth-analytics-python/03-into-to-lidar-and-raster/lidar-raster-intro/2018-02-05-raster07-crop-raster_5_0.png">

</figure>




## Open Vector Layer

Next, let's open up a vector layer that contains the crop extent that you want
to use to crop your data. To open a shapefile you use the `gpd.read_file()` function
from geopandas.

{:.input}
```python
# open crop extent
crop_extent = gpd.read_file('data/colorado-flood/spatial/boulder-leehill-rd/clip-extent.shp')
# crop_extent = crop_extent.to_crs(epsg='32613')
```

Next, let's explore the coordinate reference system (CRS) of both of your datasets. 
Remember that in order to perform any analysis with these two datasets together,
they will need to be in the same CRS. 

{:.input}
```python
print('crop extent crs: ', crop_extent.crs)
print('lidar crs: ', lidar_chm.crs)
```

{:.output}
    crop extent crs:  {'init': 'epsg:32613'}
    lidar crs:  CRS({'init': 'epsg:32613'})



{:.input}
```python
# plotting with cartopy
crs = cp.crs.epsg('32613')
fig, ax = plt.subplots(subplot_kw={'projection': crs}) 
ax.add_geometries(crop_extent['geometry'], crs=crs)
ax.set(xlim=crop_extent.bounds[['minx', 'maxx']].values[0],
       ylim=crop_extent.bounds[['miny', 'maxy']].values[0])
```

{:.output}
{:.execute_result}



    [(4434000.0, 4436000.0), (472510.46511627914, 476010.46511627914)]





{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}//images/courses/earth-analytics-python/03-into-to-lidar-and-raster/lidar-raster-intro/2018-02-05-raster07-crop-raster_10_1.png">

</figure>




{:.input}
```python
# Or use matplotlib
fig, ax = plt.subplots(figsize = (6, 6))
crop_extent.plot(ax=ax)
ax.set_title("Shapefile imported into Python - crop extent", fontsize = 16);
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}//images/courses/earth-analytics-python/03-into-to-lidar-and-raster/lidar-raster-intro/2018-02-05-raster07-crop-raster_11_0.png">

</figure>




<figure>
    <a href="{{ site.url }}/images/courses/earth-analytics/spatial-data/spatial-extent.png">
    <img src="{{ site.url }}/images/courses/earth-analytics/spatial-data/spatial-extent.png" alt="The spatial extent of a shapefile the geographic edge or location that is the furthest north, south east and west."></a>

    <figcaption>The spatial extent of a shapefile represents the geographic "edge" or location that is the furthest north, south east and west. Thus is represents the overall geographic coverage of the spatial
    object. Image Source: Colin Williams, NEON.
    </figcaption>
</figure>



Now that you have imported the shapefile. You can use the `crop_extent` function from `matplotlib` to crop the raster data using the vector shapefile.

{:.input}
```python
bounds = lidar_chm.bounds
bounds = [bounds.left, bounds.right, bounds.bottom, bounds.top]
```

{:.input}
```python
fig, ax = plt.subplots()
ax.imshow(lidar_chm_im, cmap='terrain', extent=bounds)
crop_extent.plot(ax=ax, alpha=.8);
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}//images/courses/earth-analytics-python/03-into-to-lidar-and-raster/lidar-raster-intro/2018-02-05-raster07-crop-raster_14_0.png">

</figure>




{:.input}
```python
crop_bounds = crop_extent.total_bounds
```

{:.input}
```python
fig, ax = plt.subplots()
im = ax.imshow(lidar_chm_im, cmap='terrain', extent=bounds)
ax.set(xlim=[crop_bounds[0], crop_bounds[2]], ylim=[crop_bounds[1], crop_bounds[3]])
crop_extent.plot(ax=ax, linewidth=3, alpha=.5);

# NOTE: you only need to use cartopy to use `add_geometries`.
# If you don't need to use that method, you can just use geopandas/vanilla matplotlib
# ax.add_geometries(crop_extent['geometry'], crs=cp.crs.PlateCarree())
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}//images/courses/earth-analytics-python/03-into-to-lidar-and-raster/lidar-raster-intro/2018-02-05-raster07-crop-raster_16_0.png">

</figure>




If you want to crop the data itself, rather than simply to draw a smaller area in the vizualization, you can use the `mask` function in `rasterio`.

{:.input}
```python
from rasterio.mask import mask
from shapely.geometry import mapping
```

{:.input}
```python
with rio.open("data/colorado-flood/spatial/boulder-leehill-rd/outputs/lidar_chm.tif") as lidar_chm:
    #lidar_chm_im = lidar_chm.read(masked = True)[0]
    extent_geojson = mapping(crop_extent['geometry'][0])
    lidar_chm_crop, _ = mask(lidar_chm, [extent_geojson], crop=True)
```

{:.input}
```python
fig, ax = plt.subplots()
ax.imshow(lidar_chm_crop[0], extent=bounds)
ax.set_axis_off()
```

{:.output}
{:.display_data}

<figure>

<img src = "{{ site.url }}//images/courses/earth-analytics-python/03-into-to-lidar-and-raster/lidar-raster-intro/2018-02-05-raster07-crop-raster_20_0.png">

</figure>




{:.input}
```python
# this is failing because it's trying to get the profile - which actually needs to be updated with the new crop extent
# i figured out how to do this i just forget now how it works...
# Save to disk so you can use later.
path_out = "data/colorado-flood/spatial/boulder-leehill-rd/outputs/lidar_chm_cropped.tif"
with rio.open(path_out, 'w', **lidar_chm.profile) as ff:
    ff.write(lidar_chm_crop[0], 1)
```

{:.output}

    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    <ipython-input-20-8e465f0f00f1> in <module>()
          1 # Save to disk so you can use later.
          2 path_out = "data/colorado-flood/spatial/boulder-leehill-rd/outputs/lidar_chm_cropped.tif"
    ----> 3 with rio.open(path_out, 'w', **lidar_chm.profile) as ff:
          4     ff.write(lidar_chm_crop[0], 1)


    rasterio/_base.pyx in rasterio._base.DatasetReader.profile.__get__ (rasterio/_base.c:11606)()


    rasterio/_base.pyx in rasterio._base.DatasetReader.is_tiled.__get__ (rasterio/_base.c:11120)()


    rasterio/_base.pyx in rasterio._base.DatasetReader.block_shapes.__get__ (rasterio/_base.c:6528)()


    ValueError: can't read closed raster file



<div class="notice--warning" markdown="1">

## <i class="fa fa-pencil-square-o" aria-hidden="true"></i> Optional Challenge: Crop Change Over Time Layers

In the previous lesson, you created 2 plots:

1. A classified raster map that shows **positive and negative change** in the canopy
height model before and after the flood. To do this you will need to calculate the
difference between two canopy height models.
2. A classified raster map that shows **positive and negative change** in terrain
extracted from the pre and post flood Digital Terrain Models before and after the flood.

Create the same two plots except this time CROP each of the rasters that you plotted
using the shapefile: `data/week-03/boulder-leehill-rd/crop_extent.shp`

For each plot, be sure to:

* Add a legend that clearly shows what each color in your classified raster represents.
* Use proper colors.
* Add a title to your plot.

You will include these plots in your final report due next week.
</div>
