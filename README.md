# Watershed Delineation
A simple and efficient water delineation simulation in Python which utilizes a digital elevation machine learning model.

## What is Watershed Delineation?
**Watershed delineation** is a process for creating a boundary that represents the contributing area for a specific control point or water outlet, with the intent of characterization and analysis of portions of a study area.

## Example Usage
Example data used in this tutorial are linked below:

- Elevation: [elevation.tiff](https://pysheds.s3.us-east-2.amazonaws.com/data/elevation.tiff)
- Terrain: [impervious_area.zip](https://pysheds.s3.us-east-2.amazonaws.com/data/impervious_area.zip)
- Soil Polygons: [soils.zip](https://pysheds.s3.us-east-2.amazonaws.com/data/soils.zip)

Additional DEM datasets are available via the [USGS HydroSHEDS](https://www.hydrosheds.org/) project.

### Read DEM Data
```ruby
# Read elevation raster
# ----------------------------
from pysheds.grid import Grid

grid = Grid.from_raster('elevation.tiff')
dem = grid.read_raster('elevation.tiff')
```

Plotting Code:
```ruby
import numpy as np
import matplotlib.pyplot as plt
from matplotlib import colors
import seaborn as sns

fig, ax = plt.subplots(figsize=(8,6))
fig.patch.set_alpha(0)

plt.imshow(dem, extent=grid.extent, cmap='terrain', zorder=1)
plt.colorbar(label='Elevation (m)')
plt.grid(zorder=0)
plt.title('Digital elevation map', size=14)
plt.xlabel('Longitude')
plt.ylabel('Latitude')
plt.tight_layout()
```

![Image not found](https://github.com/aryaninamdar/Watershed-Delineation/blob/main/examples/example1.png)

### Condition the Elevation Data
```ruby
# Condition DEM
# ----------------------
# Fill pits in DEM
pit_filled_dem = grid.fill_pits(dem)

# Fill depressions in DEM
flooded_dem = grid.fill_depressions(pit_filled_dem)
    
# Resolve flats in DEM
inflated_dem = grid.resolve_flats(flooded_dem)
```

### Elevation to Flow Direction
```ruby
# Determine D8 flow directions from DEM
# ----------------------
# Specify directional mapping
dirmap = (64, 128, 1, 2, 4, 8, 16, 32)
    
# Compute flow directions
# -------------------------------------
fdir = grid.flowdir(inflated_dem, dirmap=dirmap)
```

Plotting Code:
```ruby
fig = plt.figure(figsize=(8,6))
fig.patch.set_alpha(0)

plt.imshow(fdir, extent=grid.extent, cmap='viridis', zorder=2)
boundaries = ([0] + sorted(list(dirmap)))
plt.colorbar(boundaries= boundaries,
             values=sorted(dirmap))
plt.xlabel('Longitude')
plt.ylabel('Latitude')
plt.title('Flow direction grid', size=14)
plt.grid(zorder=-1)
plt.tight_layout()
```
