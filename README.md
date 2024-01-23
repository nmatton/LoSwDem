# Line-of-Sight-Analysis-in-Digital-Elevation-Models-using-Python

This tutorial demonstrates a line-of-sight (LOS) analysis algorithm based on Digital Elevation Models (DEMs) in python. The script processes elevation data to determine the visibility between two points, assessing terrain obstruction along the direct line connecting them. The implementation is explained, and potential applications of LOS analysis are telecommunications, urban planning, and environmental monitoring.

To get full source code for this tutorial, click here.
Step-by-Step Tutorial for Line of Sight Analysis Code
This code performs line of sight (LOS) analysis on a digital elevation model (DEM). It determines which pixels in the DEM are visible from a given transmitter location.

**1. Import Libraries and Define Functions:**

Import necessary libraries: rasterio, matplotlib, numpy, math.
Define a function is_visible to check if a point is visible from the transmitter.

**2. Load DEM Data:**

Open the DEM file using rasterio.open and read the data into an array.
Extract the transformation matrix for coordinate conversion.
Plot the DEM using imshow and colorbar.

**3. Visibility Function Explanation:**

The is_visible function takes the DEM array, transmitter coordinates, and target pixel coordinates as input.
It calculates the number of steps needed based on the greater of dx or dy.
It iterates over the steps, incrementing x and y coordinates gradually.
At each step, it compares the DEM elevation with the line elevation calculated using the transmitter height and target elevation.
If the DEM elevation is higher than the line elevation, the line of sight is blocked, and the function returns False.

**4. Calculate Visibility for Each Pixel**:

Define the transmitter coordinates.
Convert the latitude and longitude coordinates to grid coordinates using the inverse transform of the DEM.
Create a new empty array to store visibility values.
Loop through each pixel in the DEM array.
Call the is_visible function for each pixel with the transmitter coordinates and the current pixel coordinates.
Store the returned value (True or False) in the visibility array.

**5. Save Visibility Results and Display:**

Open a new file to write the visibility data using rasterio.
Write the visibility array to the new file.
Plot the visibility array using imshow and colorbar.
Output of LOS raster:

![image](https://github.com/nmatton/LoSwDem/assets/6585327/162c585c-3a08-405c-8a4f-65bed03d7eff)


**Note:**

This code assumes that the DEM data is in the GeoTIFF format and uses the WGS84 coordinate reference system.
The code does not account for obstacles other than the elevation data.
The code can be adapted to different scenarios by modifying the is_visible function and adding additional features.
This tutorial provides a basic understanding of the code and its functionalities. For further details, please refer to the code itself and the documentation of the libraries used.


```python
import rasterio
from rasterio.plot import show
import matplotlib.pyplot as plt
import numpy as np
import math
 
# Replace this with the path to your DEM file
# Load the DEM data
dem_data = rasterio.open('data/input/dem.tif')
dem_array = dem_data.read(1)
transform = dem_data.transform
 
# Plot DEM data
plt.imshow(dem_array, cmap='gray')
plt.colorbar()
plt.title('Digital Elevation Model')
plt.show()
 
 
def is_visible(dem_array, x1, y1, x2, y2):
    """Determine if the point (x2, y2) is visible from (x1, y1) in dem_array."""
 
    # Calculate the number of steps for iteration based on the greater of dx or dy
    dx = x2 - x1
    dy = y2 - y1
    steps = max(abs(dx), abs(dy))
 
    # Handle the case when steps is zero
    if steps == 0:
        return False
 
    # Calculate the increments for x and y
    x_inc = dx / steps
    y_inc = dy / steps
 
    # Start from the source point
    x = x1
    y = y1
 
    # Height at the source point
    z = dem_array[int(y1), int(x1)] + 20
 
    # Iterate over the cells between source and target
    for i in range(1, int(steps)):
        x += x_inc
        y += y_inc
 
        # Calculate the elevation of the direct line at this point
        line_elevation = z + (dem_array[int(y2), int(x2)] - z) * (i / steps)
 
        # If the DEM elevation at this point is higher than the direct line, it's not visible
        if dem_array[int(y), int(x)] > line_elevation:
            return False
 
    return True
 
 
# Define transmitter coordinates
transmitter_x, transmitter_y = 76.611022, 31.514496  # Example coordinates
 
 
# Transforming lat long into grid coordinates
inv_transform = ~transform
transmitter_x, transmitter_y = [
    int(round(coord)) for coord in inv_transform * (transmitter_x, transmitter_y)]
 
# Create an empty array for visibility
visibility_array = np.zeros_like(dem_array)
 
# Check visibility for each pixel
for x in range(dem_array.shape[1]):
    for y in range(dem_array.shape[0]):
        visibility_array[y, x] = is_visible(
            dem_array, transmitter_x, transmitter_y, x, y)
 
 
with rasterio.open('data/result/los.tif', 'w', **dem_data.meta) as dst:
    dst.write(visibility_array, 1)
 
plt.imshow(visibility_array, cmap='gray')
plt.colorbar()
plt.title('Line of Sight Analysis')
plt.show()
```
The provided code is not using the Bresenham Line Algorithm directly. Instead, it uses a simplified approach to check for line-of-sight visibility between two points in a Digital Elevation Model (DEM).

In the Bresenham Line Algorithm, the primary purpose is to efficiently rasterize a line between two points in a grid, considering only integer coordinates. The algorithm is widely used for drawing lines on pixel-based displays or grids.

In the provided code:

- The is_visible function iterates over the cells between the source and target points, linearly interpolating elevations along the path.
- It checks whether the elevation at each intermediate point is below the elevation of the direct line connecting the source and target. If the elevation at any point is higher, it considers the line of sight blocked.

While the iterative process in is_visible has some similarity to Bresenham’s idea of iterating over pixels, the specific logic and purpose differ. Bresenham’s Line Algorithm is primarily focused on drawing lines efficiently in a discrete grid, while the provided code is checking for line-of-sight visibility in a continuous space using linear interpolation.

source : https://spatial-dev.guru/2023/12/10/line-of-sight-analysis-in-digital-elevation-models-using-python/
