# Population Served by Public Transit & Coverage Gap Analysis (QGIS & Eurostat GEOSTAT)

![QGIS](https://img.shields.io/badge/QGIS-3.34_LTR-588240?logo=qgis&logoColor=white)
![Data Standard](https://img.shields.io/badge/Demographics-Eurostat_GEOSTAT_1km-blue)
![CRS](https://img.shields.io/badge/Projection-EPSG%3A3035%20(ETRS89)-003399)

## Project Overview
This repository presents a spatial equity and transit coverage assessment in Brussels, evaluating the proportion of residents living within a **500-meter pedestrian walking catchment (6-minute walk)** of public transit stops.

By intersecting Eurostat GEOSTAT $1\text{ km}^2$ population grids (and JRC GHSL $100\text{ m}$ downscaled rasters) with network-constrained service area hulls, the analysis quantifies urban transit accessibility and maps **Coverage Gaps**—identifying densely populated residential zones lacking adequate transit access.

## Objectives
1. **Demographic Grid Integration:** Ingest Eurostat GEOSTAT population vector grids (`EPSG:3035`) and filter for the target municipality extent.
2. **Spatial Overlay Analysis:** Perform spatial intersections (**Vector Overlay -> Intersection / Clip**) between $500\text{ m}$ pedestrian network catchments and population cells.
3. **Areal Weighting & Disaggregation:** Apply proportional area-weighting expressions to estimate population served in partially covered grid cells:
   $$\text{Pop}_{\text{Served}} = \text{Pop}_{\text{Cell}} \times \left( \frac{\text{Area}_{\text{Intersection}}}{\text{Area}_{\text{Cell}}} \right)$$
4. **Coverage Gap Identification:** Isolate residential grid cells with high population density ($> 1,500\text{ inh/km}^2$) that have less than $50\%$ coverage by $500\text{ m}$ service catchments.
5. **Cartographic Gap Mapping:** Produce a publication-ready A3 map illustrating populated grid cells, served zones, and highlighted priority transit gaps.

---
```
## Spatial Overlay Workflow Architecture
+-----------------------------------+       +-----------------------------------+
|  GEOSTAT 1km² Population Grid     |       |  500m Network Catchments (Day 5)  |
|  (Vector Polygons / TOT_P)        |       |  (Concave Hull Polygons)          |
+-----------------------------------+       +-----------------------------------+
     \                                           /
      \                                        /
       v                                      v
+---------------------------------------------+
|   QGIS Intersection / Union Algorithm       |
+---------------------------------------------+
                      |
                      v
+---------------------------------------------+
| Proportional Areal Weighting Expression     |
|  pop_served = TOT_P * ($area / cell_area)   |
+---------------------------------------------+
                      |
                      v
+---------------------------------------------+
| Summary Table & Coverage Gap Map Generation |
+---------------------------------------------+

---
```
## Workflow Implementation

### Step 1: Population Grid & Catchment Preparation
* **QGIS Manual Reference:** *Section 3.3 - Vector Selection & Filtering*
* Loaded Eurostat GEOSTAT population grid layer and clipped to Brussels municipal administrative boundaries (`LAU`).
* Filtered cells with `TOT_P > 0` to exclude unpopulated industrial or forest zones.

### Step 2: Vector Overlay & Intersect Processing
* **QGIS Manual Reference:** *Section 6.2.8 - Spatial Overlay (Intersection)*
* Executed **Vector Overlay -> Intersection**:
  * **Input Layer:** `geostat_grid_population`
  * **Overlay Layer:** `catchment_500m_network`
* Added field `pop_served` via Field Calculator:
  ```sql
  "TOT_P" * ($area / "orig_cell_area")
  ```
### Step 3: Coverage Gap Classification
* **QGIS Manual Reference:** *Section 8.2 - Raster/Vector Spatial Statistics*
* **Identified underserved zones by calculating coverage_ratio = pop_served / TOT_P:**
    *Fully Served (>80% coverage): High-density urban core.*
    *Partially Served (50% - 80% coverage): Suburban transition zones.*
    *Transit Coverage Gap (<50% coverage & TOT_P > 1,000): High-priority planning intervention targets.*
