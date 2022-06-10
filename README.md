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

![Image not found](https://github.com/aryaninamdar/Watershed-Delineation/blob/main/examples/example2.png)

### Compute Accumulation From Flow Direction
```ruby
# Calculate flow accumulation
# --------------------------
acc = grid.accumulation(fdir, dirmap=dirmap)
```

Plotting Code:
```ruby
fig, ax = plt.subplots(figsize=(8,6))
fig.patch.set_alpha(0)
plt.grid('on', zorder=0)
im = ax.imshow(acc, extent=grid.extent, zorder=2,
               cmap='cubehelix',
               norm=colors.LogNorm(1, acc.max()),
               interpolation='bilinear')
plt.colorbar(im, ax=ax, label='Upstream Cells')
plt.title('Flow Accumulation', size=14)
plt.xlabel('Longitude')
plt.ylabel('Latitude')
plt.tight_layout()
```

![Image not found](https://github.com/aryaninamdar/Watershed-Delineation/blob/main/examples/example3.png)

### Delineate Catchment From Flow Direction
```ruby
# Delineate a catchment
# ---------------------
# Specify pour point
x, y = -97.294, 32.737

# Snap pour point to high accumulation cell
x_snap, y_snap = grid.snap_to_mask(acc > 1000, (x, y))

# Delineate the catchment
catch = grid.catchment(x=x_snap, y=y_snap, fdir=fdir, dirmap=dirmap, 
                       xytype='coordinate')

# Crop and plot the catchment
# ---------------------------
# Clip the bounding box to the catchment
grid.clip_to(catch)
clipped_catch = grid.view(catch)
```

Plotting Code:
```ruby
# Plot the catchment
fig, ax = plt.subplots(figsize=(8,6))
fig.patch.set_alpha(0)

plt.grid('on', zorder=0)
im = ax.imshow(np.where(clipped_catch, clipped_catch, np.nan), extent=grid.extent,
               zorder=1, cmap='Greys_r')
plt.xlabel('Longitude')
plt.ylabel('Latitude')
plt.title('Delineated Catchment', size=14)
```

![Image not found](https://github.com/aryaninamdar/Watershed-Delineation/blob/main/examples/example4.png)

### Extract the River Network
```ruby
# Extract river network
# ---------------------
branches = grid.extract_river_network(fdir, acc > 50, dirmap=dirmap)
```

Plotting Code:
```ruby
sns.set_palette('husl')
fig, ax = plt.subplots(figsize=(8.5,6.5))

plt.xlim(grid.bbox[0], grid.bbox[2])
plt.ylim(grid.bbox[1], grid.bbox[3])
ax.set_aspect('equal')

for branch in branches['features']:
    line = np.asarray(branch['geometry']['coordinates'])
    plt.plot(line[:, 0], line[:, 1])
    
_ = plt.title('D8 channels', size=14)
```
