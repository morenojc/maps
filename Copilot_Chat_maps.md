# Copilot Chat - Maps Project
**Date:** September 29, 2025  
**Project:** Southampton Common GPS Analysis

## Overview
This chat documents the development of a Python solution to read GPS data from a GPX file, create coordinate transformations, generate polygons for boundary detection, and calculate area measurements for Southampton Common park.

## Tasks Completed

### 1. Initial GPX File Reading
**User Request:** Create Python code to read "Southampton_Common.gpx" and save coordinates into a DataFrame named `Common_Park_Coord` with latitude and longitude columns.

**Solution Implemented:**
```python
import pandas as pd
import xml.etree.ElementTree as ET

# Read and parse GPX file
with open('Southampton_Common.gpx', 'r') as file:
    gpx_content = file.read()

root = ET.fromstring(gpx_content)
namespace = {'gpx': 'http://www.topografix.com/GPX/1/1'}

# Extract coordinates from track points
coordinates = []
for trkpt in root.findall('.//gpx:trkpt', namespace):
    lat = float(trkpt.get('lat'))
    lon = float(trkpt.get('lon'))
    coordinates.append({'latitude': lat, 'longitude': lon})

Common_Park_Coord = pd.DataFrame(coordinates)
```

**Results:** Successfully loaded 336 coordinate points from Southampton Common.

### 2. Coordinate Transformation and Visualization
**User Request:** Transform coordinates and create visualizations.

**Solution Implemented:**
- Converted latitude/longitude to Euclidean coordinates with origin at south-west corner
- Created scatter plots with proper aspect ratio (`plt.axis('equal')`)
- Adjusted marker sizes for better visibility

```python
# Transform to Euclidean coordinates
Common_Park_Coord['x'] = 100 * (Common_Park_Coord['longitude'] - Common_Park_Coord['longitude'].min())
Common_Park_Coord['y'] = 100 * (Common_Park_Coord['latitude'] - Common_Park_Coord['latitude'].min())

# Plot with improved settings
plt.figure(figsize=(10, 10))
plt.scatter(Common_Park_Coord['x'], Common_Park_Coord['y'], alpha=1.0, s=5)
plt.axis('equal')  # Ensures proper square aspect ratio
```

### 3. Polygon Creation and Boundary Detection
**User Request:** Create a polygon connecting the dots with parameters for inside/outside detection functions.

**Initial Approach - Convex Hull:**
```python
from scipy.spatial import ConvexHull
hull = ConvexHull(points)
park_polygon_vertices = points[hull.vertices]
park_polygon_path = Path(park_polygon_vertices)
```

**Improved Approach - Path-Following Polygon:**
```python
# Use every nth point to follow actual GPS track
step_size = max(1, len(points) // 40)
simplified_points = points[::step_size]
# Ensure closed polygon
if not np.allclose(simplified_points[0], simplified_points[-1]):
    simplified_points = np.vstack([simplified_points, simplified_points[0]])
```

**Results:**
- Original convex hull: 8 vertices (too simplified)
- Improved polygon: 43 vertices (follows actual park boundary)

### 4. Point-in-Polygon Functions
**User Request:** Create functions to determine if points are inside or outside the park.

**Functions Implemented:**
```python
def is_point_in_park(x, y):
    """Check if single point is inside park boundary"""
    point = np.array([x, y])
    is_inside = park_polygon_path.contains_point(point)
    return "inside" if is_inside else "outside"

def check_points_in_park(x_coords, y_coords):
    """Check multiple points at once"""
    points = np.column_stack((x_coords, y_coords))
    inside_mask = park_polygon_path.contains_points(points)
    return ["inside" if inside else "outside" for inside in inside_mask]
```

**Test Results:**
- Center point (0.957, 0.947): inside
- Outside point (2.414, 2.395): outside
- Min boundary point (0.000, 0.000): outside

### 5. Area Calculation
**User Request:** Compute the area of the polygon using `park_polygon_path`.

**Solution Implemented:**
```python
def compute_polygon_area(vertices):
    """Compute area using Shoelace formula"""
    x = vertices[:, 0]
    y = vertices[:, 1]
    area = 0.5 * abs(sum(x[i] * y[i+1] - x[i+1] * y[i] for i in range(-1, len(x)-1)))
    return area

# Convert to real-world units
lat_avg = Common_Park_Coord['latitude'].mean()
lon_scale_factor = np.cos(np.radians(lat_avg))
meters_per_x_unit = 0.01 * 111000 * lon_scale_factor
meters_per_y_unit = 0.01 * 111000
```

**Results:**
- **Coordinate units:** 1.838603 square units
- **Square meters:** 1,427,802 m²
- **Hectares:** 142.78 ha
- **Acres:** 352.81 acres

## Technical Details

### Libraries Used
- `pandas`: DataFrame operations
- `xml.etree.ElementTree`: GPX file parsing
- `matplotlib.pyplot`: Visualization
- `matplotlib.patches.Polygon`: Polygon creation
- `matplotlib.path.Path`: Point-in-polygon testing
- `numpy`: Numerical operations
- `scipy.spatial.ConvexHull`: Initial polygon approach

### Coordinate System
- Origin: South-west corner of the GPS track
- Scale: 100x magnification (1 unit = 0.01 degrees)
- X-axis: Longitude (east-west)
- Y-axis: Latitude (north-south)

### Key Parameters Stored
- `park_polygon_vertices`: NumPy array with 43 polygon vertices
- `park_polygon_path`: Matplotlib Path object for efficient point-in-polygon testing
- `Common_Park_Coord`: Pandas DataFrame with original and transformed coordinates

## Visualizations Created
1. **Scatter Plot**: GPS points with proper aspect ratio
2. **Polygon Comparison**: Convex hull vs. path-following polygon
3. **Area Visualization**: Filled polygon showing park boundary and area

## Files Created
- `Testing_maps.ipynb`: Jupyter notebook with all analysis
- `Copilot_Chat_maps.md`: This chat summary

## Key Improvements Made
1. **Plotting Enhancement**: Added `plt.axis('equal')` for proper aspect ratio
2. **Marker Size**: Increased from s=1 to s=5 for better visibility
3. **Polygon Accuracy**: Improved from 8-vertex convex hull to 43-vertex path-following polygon
4. **Real-World Conversion**: Accurate area calculation in hectares and acres

## Usage Instructions
1. Load GPX file using the parsing code
2. Transform coordinates to Euclidean system
3. Use `is_point_in_park(x, y)` to test individual points
4. Use `check_points_in_park(x_coords, y_coords)` for multiple points
5. Polygon parameters are available in `park_polygon_vertices` and `park_polygon_path`

## Final Results Summary
- **GPS Points Processed:** 336 coordinates
- **Polygon Vertices:** 43 (optimized for accuracy vs. efficiency)
- **Park Area:** 142.78 hectares (352.81 acres)
- **Functions Available:** Point-in-polygon testing, area calculation
- **Coordinate System:** Euclidean transformation with proper scaling

This implementation provides a complete solution for GPS boundary detection and area analysis for Southampton Common park.
