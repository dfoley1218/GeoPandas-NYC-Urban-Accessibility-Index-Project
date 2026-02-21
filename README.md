# GeoPandas NYC Urban Accessibility Index Project

This is a geospatial analysis project built in Python utilizing GeoPandas, based on the data sets available from https://opendata.cityofnewyork.us/

---

## Project Overview

This project builds a Urban Accessibility Index for New York City by integrating multiple real-world geospatial datasets into a single composite score that quantifies how accessible each part of the city is in terms of schools, transit, cycling infrastructure, and green space. It serves as a portfolio project demonstrating proficiency in GeoPandas for spatial data analysis, visualization, and processing — spanning two levels of spatial granularity: traditional neighborhood polygons and an H3 hexagonal grid.

<img width="1492" height="602" alt="image" src="https://github.com/user-attachments/assets/80207753-40f1-4c0d-8ca4-ffe84637e215" />
<img width="1494" height="600" alt="image" src="https://github.com/user-attachments/assets/b57b53ad-bbc4-4ef0-96e3-8453ac95ce47" />

## What the Analysis Does

Five datasets are loaded and unified into a single coordinate reference system (EPSG:3857 — Web Mercator) to enable accurate metric calculations in meters:

- NYC School Points (Shapefile) — point geometries for all public school locations
- NYC Subway Entrances (Shapefile) — point geometries for every subway entrance across the five boroughs
- NYC Bike Routes (GeoJSON) — LineString geometries representing the city's cycling network
- NYC Parks Properties (GeoJSON) — Polygon geometries for all parks and green spaces
- Custom NYC Neighborhoods (GeoJSON, sourced from HodgesWardElliott/PediaCities) — Polygon boundaries for ~260 named neighborhoods across all five boroughs

## Phase 1 — Neighborhood-Level Analysis

For each neighborhood polygon, four spatial metrics are computed via geometric intersection queries:

- School count — the number of school points that intersect the neighborhood boundary
- Subway entrance count — the number of subway entrance points that intersect
- Bike path length — the total length (in meters) of bike route segments intersecting the neighborhood
- Park area — the total area (in square meters) of park polygons intersecting the neighborhood

These four raw metrics are on entirely different scales, so MinMaxScaler from scikit-learn is applied to normalize each to the [0, 1] range. The normalized values then go through a spatial smoothing step: for each neighborhood, the scores of all geometrically touching (adjacent) neighborhoods are averaged together and substituted in. This smoothing is a form of spatial lag that reduces sharp score discontinuities at neighborhood boundaries and produces a more realistic representation of accessibility — reflecting the fact that residents near the edge of one neighborhood functionally share amenities with neighboring areas. The smoothed values are normalized a second time to restore the [0, 1] scale, and a final index_score is derived by summing all four normalized dimensions. The result is exported as neighborhood_access_index.geojson.

## Phase 2 — H3 Hexagonal Grid Analysis

The neighborhood-level output is then reprojected to WGS84 (EPSG:4326) and tessellated using Uber's H3 hierarchical hexagonal grid at resolution 9 (each hexagon covers roughly 0.1 km²). The h3pandas library's polyfill method fills each neighborhood polygon with the H3 cell indices that fall within it, producing a fine-grained hexagonal mesh over the entire city. Each hexagon inherits its parent neighborhood's attributes and is then re-projected back to EPSG:3857 for metric calculation.
The same four spatial metrics are recomputed at the hexagon level — counting schools, subway entrances, measuring bike path length, and park area per hexagon via intersection. This is then normalized, spatially smoothed using H3's grid_ring function (averaging values across a 2-ring neighborhood of surrounding hexagons — roughly the 18 cells within two hexagonal steps), re-normalized, and summed into a final hexagon-level index_score. The result is exported as access_index.geojson.

## Visualization

Both the neighborhood-level and hexagon-level index scores are visualized as interactive choropleth maps using Leafmap, with a quantile classification scheme and a Blues colormap, allowing for exploration of accessibility variation across the five boroughs at two different spatial resolutions.

## Why Two Spatial Scales?

The dual-resolution approach illustrates a core challenge in spatial analysis: the Modifiable Areal Unit Problem (MAUP). Neighborhood polygons are irregularly shaped and vary widely in size — a single score per neighborhood smooths over a lot of internal variation. The H3 grid imposes a uniform, consistent unit of analysis across the entire city, making scores more directly comparable and revealing fine-grained accessibility gradients that neighborhood-level aggregation would otherwise hide.

## Topics Covered

- GeoDataFrames, spatial joins, and geometry manipulations — loading, reprojecting, and querying multi-format vector datasets with GeoPandas and Shapely
- Handling vector data, projections, and common file formats — working across Shapefiles, GeoJSON, and GeoParquet; managing CRS transformations between EPSG:3857 and EPSG:4326
- Spatial queries, intersections, and overlays — using intersects and touches predicates to extract neighborhood-level metrics from point, line, and polygon datasets
- H3 hexagonal indexing — tessellating irregular polygon boundaries with Uber's H3 grid using h3pandas, performing neighbor ring lookups with h3.grid_ring, and working with H3 cell indices as GeoDataFrame indices
- Spatial smoothing and the MAUP — implementing adjacency-based and ring-based spatial lag smoothing to reduce boundary artefacts at two different scales
- Feature normalization — applying MinMaxScaler from scikit-learn across multiple rounds of normalization to produce a meaningful composite index
- Interactive geospatial visualization — rendering quantile choropleth maps with Leafmap and exporting analysis outputs to GeoJSON for downstream use


---

## Tech Stack

- **Python 3.x**
- **GeoPandas** – core geospatial data library
- **Pandas** – data manipulation and tabular operations
- **Shapely** – geometric operations (Point, Polygon, intersection predicates)
- **h3pandas** – H3 hexagonal grid integration with GeoDataFrames
- **h3** – hexagonal neighbor ring lookups (`grid_ring`)
- **scikit-learn** – feature normalization via `MinMaxScaler`
- **NumPy** – numerical computing
- **Leafmap** – interactive choropleth map visualization
- **Fiona** – spatial file I/O (used by GeoPandas for Shapefile and GeoJSON support)
- **Jupyter Notebook** – interactive development environment

---

## Getting Started

### 1. Clone the Repository
```bash
git clone https://github.com/dfoley1218/GeoPandas-NYC-Project.git
cd GeoPandas-NYC-Project
```

### 2. Install Dependencies
```bash
pip install geopandas pandas matplotlib shapely fiona jupyter
```
> **Recommended:** Use `conda` to avoid dependency conflicts:
> ```bash
> conda install -c conda-forge geopandas
> ```

### 3. Launch the Notebook
```bash
jupyter notebook
```

---

## Project Structure

```
GeoPandas-NYC-Project/
│
├── data/               # Source data (Shapefiles, GeoJSON, GeoParquet)
├── notebooks/          # Jupyter notebooks with analysis
├── outputs/            # Generated maps and exported files
└── README.md
```

---

##  References

### Data Sources

| Dataset | Source | Format |
|---|---|---|
| [School Point Locations](https://data.cityofnewyork.us/Education/School-Point-Locations/jfju-ynrr) | NYC Open Data – NYC Dept. of Education | Shapefile |
| [NYC Transit Subway Entrance & Exit Data](https://data.ny.gov/Transportation/NYC-Transit-Subway-Entrance-And-Exit-Data/i9wp-a4ja) | NY State Open Data – MTA | Shapefile |
| [New York City Bike Routes](https://data.cityofnewyork.us/dataset/New-York-City-Bike-Routes/mzxg-pwib) | NYC Open Data – NYC DOT | GeoJSON |
| [Parks Properties](https://data.cityofnewyork.us/Recreation/Parks-Properties/enfh-gkve) | NYC Open Data – NYC Dept. of Parks & Recreation | GeoJSON |
| [Custom NYC Neighborhoods](https://github.com/HodgesWardElliott/custom-nyc-neighborhoods) | HodgesWardElliott / PediaCities (GitHub) | GeoJSON |

> **Note:** The subway entrance data is hosted on **NY State Open Data** (`data.ny.gov`) via the MTA, not on NYC Open Data directly.

---

### Library Documentation

- [GeoPandas](https://geopandas.org/en/stable/docs.html)
- [h3-py](https://uber.github.io/h3-py/api_reference) – Uber's H3 hexagonal grid bindings for Python
- [h3pandas](https://h3pandas.readthedocs.io/en/latest/) – H3 integration with GeoPandas/Pandas
- [Shapely](https://shapely.readthedocs.io/en/stable/)
- [scikit-learn – MinMaxScaler](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.MinMaxScaler.html)
- [Leafmap](https://leafmap.org/)
- [Fiona](https://fiona.readthedocs.io/en/latest/)
