# SylhetSOC Project Documentation

## Project Overview

**Research Topic**: Soil Organic Carbon (SOC) Dynamics in Wetland Ecosystems
**Study Area**: Sylhet Basin, Bangladesh
**Study Period**: 1985-2025 (40 years)
**Focus**: Understanding SOC dynamics in relation to hydrodynamics, vegetation changes, and climatic variations

### Research Locations (9 sites)
1. Ajmiriganj
2. Balaganj
3. Goainghat
4. Hakaluki
5. Kanairghat
6. Phagu
7. Sarail
8. Sulla
9. Terchibari

---

## Project Structure

### Main Notebooks
- **Analysis.ipynb** - Core analysis with PCA, clustering, and correlations
- **Biomass.ipynb** - Biomass calculations from Landsat and ESA CCI data
- **EDA.ipynb** - Exploratory data analysis, 3D visualization, soil correlation
- **Geospatial.ipynb** - Google Earth Engine processing, Sentinel-2 analysis
- **Hydrological.ipynb** - Land cover changes and hydrological dynamics
- **An.ipynb** - Additional analysis notebook
- **Hy1.ipynb** - Additional hydrological analysis notebook

### Data Directories
```
SylhetSOC/
├── data/                 # Soil property CSV files
│   ├── PresentTopSoil.csv (2025 data)
│   ├── PresentSubSoil.csv (2025 data)
│   ├── PreviousTopSoil.csv (1985 data)
│   ├── PreviousSubSoil.csv (1985 data)
│   ├── TopSoil.csv (merged 1985-2025)
│   ├── MainPp.csv (previous period data)
│   ├── MainCc.csv (current period data)
│   └── GeoData.csv (geospatial reference)
│
├── geodata/              # Geospatial analysis outputs
│   ├── indices_1985_2025.csv (spectral indices)
│   ├── biomass_agb.csv (above-ground biomass)
│   ├── biomass_ndvi.csv (NDVI-derived biomass)
│   ├── ndvi_changes.csv (NDVI temporal series)
│   ├── LULCAreaCover.csv (2017-2024 land cover)
│   └── Hydro_Veg.csv (hydro-veg-SOC correlations)
│
├── gis/                  # GIS files and rasters
│   ├── StudyArea.shp/dbf/prj/shx (study boundary)
│   ├── NDVI_*.tif (NDVI by decade)
│   ├── NDWI_*.tif (water index by decade)
│   ├── BUI_*.tif (built-up index by decade)
│   ├── LULC*.tif (land use 2017-2024)
│   └── StockCIDW.tif (kriged SOC stock)
│
└── Figure/               # Output visualizations
    ├── NDVIChanges.png
    ├── NDWIChanges.png
    ├── BUIChanges.png
    ├── Hydro_*.png
    └── 1985_2025_ChangesIndices.png
```

---

## Key Calculations & Formulas

### 1. NDVI (Normalized Difference Vegetation Index)
- **Formula**: `NDVI = (NIR - Red) / (NIR + Red)`
- **Landsat 5/7 Bands**: B4 (Red), B3 (Green), B4 (NIR)
- **Landsat 8 Bands**: B4 (Red), B5 (NIR)
- **Data Source**: Google Earth Engine, Landsat Collection 2 Level 2 Surface Reflectance
- **Temporal**: Annual means, 1985-2025
- **Output**: `geodata/ndvi_changes.csv`

### 2. NDWI (Normalized Difference Water Index)
- **Formula**: `NDWI = (NIR - SWIR) / (NIR + SWIR)`
- **Purpose**: Water availability and vegetation water stress
- **Rasters**: `gis/NDWI_*.tif` (1985-1995, 1996-2005, 2006-2015, 2016-2025)

### 3. BUI (Built-Up Index)
- **Purpose**: Urban expansion monitoring
- **Data**: Landsat thermal and optical bands
- **Rasters**: `gis/BUI_*.tif` (decadal composites)

### 4. LST (Land Surface Temperature)
- **Source**: Landsat 8 thermal bands (Band 10/11)
- **Status**: Preliminary; sentinel value -9999 for pre-2010 data
- **Output**: `geodata/indices_1985_2025.csv` (mean_lst column)

### 5. Biomass Vegetation

#### NDVI-Based Biomass
```python
Biomass (t/ha/yr) = 11.59 × NDVI² - 4.96 × NDVI + 0.76
```
- **Output**: `geodata/biomass_ndvi.csv`
- **Range**: 0.28-0.55 t/ha/yr
- **Data Quality**: NaN values before 1988 (sensor availability)

#### AGB (Above-Ground Biomass)
- **Source**: ESA CCI AGB ImageCollection
- **Resolution**: ~100 meters
- **Coverage**: 2010, 2015-2021 (limited temporal coverage)
- **Values**: 20-30 Mg/ha
- **Output**: `geodata/biomass_agb.csv`

### 6. SOC (Soil Organic Carbon)
**Measurements taken at each location:**
- **SOC%**: Soil organic carbon percentage
- **SOCT**: Total SOC content
- **SOCw**: Weighted SOC
- **SOCi**: Immobilized SOC
- **Stock**: SOCT_topsoil + SOCT_subsoil

**Soil Properties also measured:**
- pH, TN (Total Nitrogen), Clay %, SBD (Soil Bulk Density), CEC (Cation Exchange Capacity)

**Important Finding**: SOC values unchanged between 1985-2025 across all locations

---

## Land Cover/Land Use (LULC) Classification

### Class Codes
```
1 = Water
2 = Vegetation
4 = Flood
7 = Urban
9 = Flooded (wetland)
```

### Temporal Coverage
- 2017-2024 annual LULC maps (8 rasters)
- File pattern: `gis/LULC{YEAR}c.tif`

### Area Change Analysis (2017-2024)
**Within 500m buffer around each site:**
- Water Area Change: **-1.8 billion m²** (dramatic loss)
- Flood Area Change: **-454 million m²** (significant reduction)
- Vegetation Change: **-244 million m²** (vegetation loss)
- Urban Area Change: **+675 million m²** (urban expansion)

---

## Key Correlations & Findings

### SOC Correlations
| Relationship | Correlation | Strength |
|---|---|---|
| Stock ↔ SOC% | 0.92 | **Very Strong** |
| TN ↔ SOC% | 0.82 | **Strong** |
| pH ↔ SOC% | -0.52 | **Moderate Negative** |
| Vegetation Loss ↔ SOC | -0.40 | **Moderate Negative** |
| Urban Expansion ↔ SOC | -0.27 | **Weak Negative** |

### PCA Analysis
- **PC1**: 40.4% variance (driven by hydrological changes)
- **PC2**: 24.8% variance (driven by vegetation and soil nutrients)
- **Total**: 65.2% variance explained by first two components

### Clustering Results
- **Best Method**: K-Means (n=2)
- **Silhouette Score**: [Check Analysis.ipynb for exact values]
- **Clusters**: Two distinct location groups based on environmental characteristics

---

## Important Observations

### 1. SOC Stability
- **No change** in SOC values between 1985 and 2025
- Suggests either: effective carbon retention OR potential data consistency issues
- **Critical for interpretation**: Verify if repeated soil sampling at exact same locations

### 2. Dramatic Environmental Changes
Despite stable SOC, significant changes in:
- Water resources (massive decline)
- Vegetation cover (loss)
- Urban pressure (expansion)
- Hydrological regime

### 3. Data Gaps
- ESA CCI AGB data limited (2010, 2015-2021)
- LST data sparse before 2010
- LULC classification only 2017-2024

### 4. Study Period Divisions
- **1985-1988**: Landsat data sparse
- **1988-2005**: Continuous Landsat coverage
- **2005-2025**: High-quality Sentinel-2/Landsat-8
- **2017-2024**: LULC detailed annual maps

---

## Google Earth Engine Processing

### Authentication
```python
ee.Authenticate()  # One-time authentication
ee.Initialize(project="ee-arsenicbd")
```

### Key GEE Assets Used
- `LANDSAT/LT05/C02/T1_L2` (Landsat 5)
- `LANDSAT/LE07/C02/T1_L2` (Landsat 7)
- `LANDSAT/LC08/C02/T1_L2` (Landsat 8)
- `COPERNICUS/S2` (Sentinel-2)
- `projects/sat-io/open-datasets/ESA/ESA_CCI_AGB`

### Processing Parameters
- **Cloud Filter**: < 30% cloud cover
- **Sentinel-2 Resolution**: 10m (visual bands)
- **Landsat Resolution**: 30m
- **Temporal Compositing**: Annual means
- **Export Format**: GeoTIFF, EPSG:4326

---

## Dependencies & Tools

### Python Libraries
- **pandas** - Data manipulation
- **geopandas** - Geospatial operations
- **rasterio** - Raster data handling
- **matplotlib/seaborn** - Visualization
- **scikit-learn** - PCA, clustering, regression
- **earthengine-api** - Google Earth Engine access
- **scipy** - Statistical analysis

### External Data Sources
- Google Earth Engine (Landsat, Sentinel-2, ESA CCI)
- Field measurements (9 sampling locations)
- Shapefile of study area

---

## For Future Work

### Recommended Analyses
1. **Temporal Regression**: Model SOC changes over time with environmental drivers
2. **Spatial Interpolation**: Create SOC distribution maps using kriging
3. **Machine Learning**: Predict SOC from spectral indices
4. **Uncertainty Analysis**: Assess sampling and measurement errors
5. **Climate Integration**: Include temperature/precipitation data

### Data Recommendations
- Verify 1985-2025 SOC measurement methodology
- Ensure consistent sampling locations across decades
- Obtain complete LST and AGB time series
- Consider finer temporal resolution for LULC (annual maps exist 2017+)

### Visualization Improvements
- Create animation showing decadal changes
- Interactive maps with overlaid indices
- Time series plots by location
- Statistical significance testing for correlations

---

## Quick Reference: File Locations

**Most Important Files**:
- Soil data: `data/TopSoil.csv` (merged 1985-2025)
- Spectral indices: `geodata/indices_1985_2025.csv`
- Correlations: `geodata/Hydro_Veg.csv`
- Land cover: `geodata/LULCAreaCover.csv`

**Raster Files for Visualization**:
- NDVI: `gis/NDVI_*.tif`
- Water index: `gis/NDWI_*.tif`
- Urban: `gis/BUI_*.tif` or LULC urban class
- SOC stock: `gis/StockCIDW.tif`

---

*Last Updated: 2025-12-12*
*Project Status: Active research on SOC dynamics in Sylhet wetland ecosystem*
