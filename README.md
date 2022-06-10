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
