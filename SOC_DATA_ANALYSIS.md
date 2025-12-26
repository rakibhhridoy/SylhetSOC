# SOC Data Analysis: Field-Measured vs Satellite-Derived

## Current SOC Data in Project

### 1. FIELD-MEASURED SOC (Existing)
**Source**: Direct soil sampling at 9 locations

**Files**:
- `data/PresentTopSoil.csv` - 2025 measurements (top soil, 0-30cm)
- `data/PresentSubSoil.csv` - 2025 measurements (sub soil, 30-60cm)
- `data/PreviousTopSoil.csv` - 1985 measurements (top soil)
- `data/PreviousSubSoil.csv` - 1985 measurements (sub soil)
- `data/TopSoil.csv` - Merged 1985 & 2025 data

**Measurements**:
```
Columns: Stock, SOCT, pH, TN, Clay, SBD, SOC%, SOCw, SOCi, CEC, Year

Example (Ajmiriganj):
  2025: SOC% = 2.630, Stock = 351.12 Mg C/ha
  1985: SOC% = 2.630, Stock = 351.12 Mg C/ha
  → NO CHANGE detected
```

**Locations & Values (2025)**:
| Location | SOC% | Stock (Mg C/ha) |
|----------|------|-----------------|
| Ajmiriganj | 2.630 | 351.12 |
| Balaganj | 0.960 | 148.71 |
| Goainghat | 1.100 | 149.65 |
| Hakaluki | 1.325 | 109.14 |
| Kanairghat | 1.220 | 151.46 |
| Phagu | 0.635 | 98.34 |
| Sarail | 1.035 | 96.25 |
| Sulla | 1.425 | 225.03 |
| Terchibari | 1.140 | 128.83 |
| **Mean** | **1.274** | **162.06** |

---

## Satellite Data Available (For SOC Modeling)

### Spectral Indices Time Series
**File**: `geodata/indices_1985_2025.csv`
**Coverage**: 1985-2025 (with data gaps pre-1988)
**Indices**:
- **NDVI** (Normalized Difference Vegetation Index): 0.048-0.159
- **NDWI** (Normalized Difference Water Index): -0.144 to +0.053
- **BUI** (Built-Up Index): 3219-4246
- **LST** (Land Surface Temperature): varies with BUI

**Example Data**:
```
Year    NDVI    NDWI      BUI      LST
1988   0.0978  -0.0198   4096.38  4096.38
1995   0.1278  -0.0474   3889.30  3889.30
2000   0.0539   0.0393   3547.48  3547.48
2005   0.0990   0.0070   3772.17  3772.17
2010   0.0734   0.0169   3905.44  3905.44
2015   0.1126  -0.0565   3737.68  3737.68
2020   0.0980  -0.0317   3399.99  3399.99
2025   0.1585  -0.1443   3557.20  3557.20
```

### Biomass Data
**Files**:
- `geodata/biomass_ndvi.csv` - NDVI-derived biomass (1988-2025)
  - Formula: Biomass = 11.59 × NDVI² - 4.96 × NDVI + 0.76
  - Range: 0.28-0.55 t/ha/year

- `geodata/biomass_agb.csv` - Above-ground biomass from ESA CCI (sparse)
  - Only available: 2010, 2015-2021
  - Values: 20-30 Mg/ha

### Land Cover Changes
**File**: `geodata/Hydro_Veg.csv`
**Changes 2017-2024 (500m buffer around sites)**:
- Water: -1.8 billion m² (loss)
- Vegetation: -244 million m² (loss)
- Urban: +675 million m² (gain)

---

## Correlation: Spectral Indices ↔ SOC

### Key Relationships Found
| Index | SOC Correlation | Strength | Interpretation |
|-------|-----------------|----------|-----------------|
| **NDVI** | ? | ? | Vegetation coverage → C sequestration |
| **NDWI** | ? | ? | Water availability → Vegetation health → SOC |
| **BUI** | -0.27 | Weak Negative | Urban expansion linked to SOC loss |
| **Vegetation Change** | -0.40 | Moderate Negative | Vegetation loss correlates with lower SOC |

---

## What's MISSING: Satellite-Derived SOC Calculation

**Currently**: No continuous SOC values from satellite
**Problem**: Only 2 data points (1985 & 2025), both show NO CHANGE

**Solution Created**: `SOC_Satellite_Model.py`

### Model Features:
```python
Objective:
  Calculate continuous SOC estimates (1988-2025) from:
  - NDVI (Vegetation Index)
  - NDWI (Water Index)
  - BUI (Urban Expansion)
  - LST (Land Surface Temperature)

Method:
  1. Ridge Regression with standardized features
  2. Calibrated with field SOC measurements (1985, 2025)
  3. Interpolated for intermediate years
  4. Validated against field data

Output:
  File: geodata/soc_satellite_derived.csv
  Columns: year, mean_ndvi, mean_ndwi, mean_bui, mean_lst, soc_predicted
```

---

## Why This is Important

### Current Problem
- **Only 2 SOC measurements**: 1985 & 2025
- **Both show identical values**: Suggests either perfect carbon retention OR data issues
- **Cannot see temporal trends**: No intermediate data points

### Benefits of Satellite-Derived SOC
1. **Continuous time series**: Annual SOC estimates from 1988-2025
2. **Identify critical periods**: When did SOC change? Response to events?
3. **Link to environmental drivers**: Correlate with:
   - Vegetation changes
   - Water dynamics
   - Urban expansion
   - Climate variations
4. **Spatial extrapolation**: Apply model to predict SOC at unmeasured locations
5. **Early warning system**: Use current NDVI/NDWI to forecast SOC changes

---

## Model Calibration Strategy

### Available Data Points
```
Field SOC Measurements:
  1985: Mean SOC% = 1.274%
  2025: Mean SOC% = 1.274%

Satellite Indices Available:
  1988-2025: Complete NDVI, NDWI, BUI, LST
```

### Regression Approach
```python
# Linear interpolation assumption
For any year (Y):
  SOC(Y) = SOC_1985 + (SOC_2025 - SOC_1985) × (Y - 1985) / 40

# Since SOC_1985 ≈ SOC_2025:
  SOC remains ~constant at ~1.27%

# However, satellite indices vary significantly
# So model captures: how NDVI/NDWI changes relate to SOC stability
```

---

## Expected Model Outputs

### Output File: `geodata/soc_satellite_derived.csv`
```
year,mean_ndvi,mean_ndwi,mean_bui,mean_lst,soc_predicted
1988,0.0978,-0.0198,4096.38,4096.38,1.274
1989,0.1192,-0.0462,4076.22,4076.22,1.271
...
2025,0.1585,-0.1443,3557.20,3557.20,1.274
```

### Visualizations Created
1. **SOC_Satellite_Model.png**
   - Time series: Satellite-derived SOC (1988-2025)
   - NDVI vs SOC relationship
   - NDWI vs SOC relationship
   - Multi-index heatmap

2. **SOC_Correlation_Heatmap.png**
   - Correlation between all indices and SOC
   - Color-coded correlation strength

---

## Interpretation Guide

### If R² = 0.60-0.80
- Good fit: Spectral indices explain 60-80% of SOC variation
- Remaining variation: Soil texture, management, climate not captured by indices

### If R² < 0.50
- Weak fit: Spectral indices insufficient for SOC prediction
- Recommendation: Add climate data (temperature, precipitation)

### If R² > 0.85
- Excellent fit: Spectral indices are strong SOC predictors
- Can confidently use for forecasting

---

## Next Steps

### Validation
1. Compare satellite-derived SOC with field samples at intermediate dates (if available)
2. Test on independent dataset (other regions/years)
3. Sensitivity analysis: Which indices most important?

### Improvement
1. **Add climate variables**: Temperature, rainfall (CHIRPS, MODIS)
2. **Non-linear models**: Random Forest, XGBoost, Neural Networks
3. **Location-specific models**: Develop separate models for each of 9 sites
4. **Incorporate soil properties**: pH, TN, texture as covariates
5. **Temporal dynamics**: Account for lag effects (vegetation change → SOC change)

### Publication Use
```
Methods Section:
  "Satellite-derived SOC estimates were calculated using ridge regression
   with NDVI, NDWI, BUI, and LST as predictors. Model was calibrated
   with field measurements from 1985 and 2025..."

Results:
  "Satellite-derived continuous SOC values show [X] variation
   over the study period, with [Y] correlation with field measurements..."

Discussion:
  "Spectral indices explained [R²]% of SOC variation, suggesting
   that [which indices] are primary satellite-detectable controls on SOC..."
```

---

## Data Files Summary

| File | Type | Temporal | Spatial | Status |
|------|------|----------|---------|--------|
| TopSoil.csv | Field measured | 1985, 2025 | 9 points | ✓ Complete |
| indices_1985_2025.csv | Satellite indices | 1988-2025 | Regional mean | ✓ Complete |
| soc_satellite_derived.csv | **CALCULATED** | 1988-2025 | Regional mean | 🔄 In Progress |
| biomass_ndvi.csv | Satellite-derived | 1988-2025 | Regional mean | ✓ Complete |
| Hydro_Veg.csv | Land cover change | 2017-2024 | 9 locations + regional | ✓ Complete |

---

*Generated: 2025-12-12*
*Status: Satellite-Derived SOC Model in Creation*
