# ML4HP-US: A Multi-Source Deep Learning–Based 4-km Hourly Precipitation Dataset for the Contiguous United States

**Status:** Pre-release; public release is planned by the end of 2026.  
**Coverage:** Contiguous United States (CONUS) | nominal 4 km | hourly | 2002–2025  
**Primary format:** NetCDF (`.nc`)  
**Approximate archive size:** 600 GB (subject to final packaging)  
**AWS availability:** Not yet published; access instructions will be added after release.

> **Documentation status:** This README describes the current production files and planned public archive. Items marked **[TO CONFIRM]** require checking against the final NetCDF metadata or approval before publication. In particular, the geodetic datum and exact time-zone/calendar metadata still require verification. The spatial coordinates are geographic latitude/longitude, not projected x/y coordinates. Hourly timestamps denote the **start** of each accumulation period, and missing precipitation values are represented by **NaN** (`np.nan`).

## 1. Overview

**ML4HP-US** is a long-term, spatially continuous, hourly precipitation dataset developed for the contiguous United States using **XTPrecip**, a multi-source spatiotemporal deep-learning framework. XTPrecip combines information from ground-based radar, satellite observations, reanalysis/land-surface products, and terrain data, and is trained using quality-screened NOAA Cooperative Observer Program (COOP) hourly precipitation observations. The modeling strategy uses CONUS-wide pretraining followed by regional fine-tuning in seven National Climate Assessment (NCA) regions.

ML4HP-US is designed to support:

- Hydrologic and land-surface model forcing;
- Flood and extreme-precipitation analysis;
- Water-resources and climate variability research;
- Assessments of radar, satellite, and reanalysis precipitation estimates; and
- Machine-learning development, benchmarking, and downscaling applications.

The currently prepared archive covers **2002–2025**, with one NetCDF file per calendar month. The archive is intended to be updated beyond 2025 after the initial public release.

## 2. Data format and organization

### File format and naming

The production archive uses **NetCDF** with hourly data stored as `float32` arrays. Each monthly file contains a `time × lat × lon` data cube.

Current filename pattern:

```text
blended_precip_YYYY_MM.nc
```

Current production directory pattern (to be retained or mapped to the public archive):

```text
blended_generation/
├── 2002/
│   ├── blended_precip_2002_01.nc
│   ├── blended_precip_2002_02.nc
│   └── ...
├── 2003/
│   └── ...
└── 2025/
    ├── ...
    └── blended_precip_2025_12.nc
```

- **Number of monthly files:** 288 (24 years × 12 months).
- **Approximate current size:** about 2 GB per monthly file, or approximately 576 GB in total; rounded to **~600 GB** for planning.
- **Final published archive size:** may change with compression, metadata, and quality-control updates.
- **Additional formats:** None are confirmed for the initial public release; cloud-oriented alternatives such as Zarr may be considered later.

## 3. Variables and metadata

### Primary variables

| Variable | Meaning | Units | Data type |
| --- | --- | --- | --- |
| `precip_raw` | XTPrecip-based blended precipitation estimate before the final intensity-scaling correction | mm h⁻¹ | `float32` |
| `precip_scaled` | Blended precipitation estimate after the intensity-scaling correction; intended primary precipitation field | mm h⁻¹ | `float32` |

Other masks or processing fields may exist in some internal versions. Only the two precipitation fields above should be treated as the currently documented public variables until the final file schema is verified.

### Dimensions and spatial grid

| Property | Current archive description |
| --- | --- |
| Dimensions | `time`, `lat`, `lon` |
| Latitude grid | 676 grid-cell centers, approximately 24.00°N to 51.00°N |
| Longitude grid | 1,500 grid-cell centers, approximately 126.00°W to 66.04°W |
| Grid spacing | 0.04° in latitude and longitude (nominally ~4 km; physical east–west spacing varies with latitude) |
| Coordinates | One-dimensional `lat` and `lon` in geographic degrees |
| Spatial coordinate system | **Geographic latitude/longitude (degrees); no map projection is used.** Coordinates are provided directly as `lat` and `lon`, not projected easting/northing (`x`/`y`). |
| Geodetic datum | **[TO CONFIRM]** from NetCDF metadata. A specific datum or EPSG code (e.g., WGS 84 / EPSG:4326) should not be asserted without verification. |
| Spatial extent versus valid area | The rectangular grid includes locations outside valid CONUS coverage; use missing-value/mask information to distinguish unavailable cells from zero rainfall |

### Time conventions

- **Temporal resolution:** hourly; nominal coverage extends from January 2002 through December 2025.
- **Time dimension:** `time` (one timestamp for each hourly field).
- **Time standard:** **[TO CONFIRM]** whether all timestamps and CF time units explicitly represent UTC in the published files.
- **Hour labeling:** Each timestamp represents the **start of the hourly precipitation accumulation interval**. For example, the record at `2020-01-01 12:00` represents precipitation accumulated from `12:00` up to (but not including) `13:00` on that date. This matters when comparing products with end-of-hour timestamps.
- **Coverage edges:** Some boundary hours are absent in the current production archive. For example, the January 2002 file begins at `2002-01-01 04:00` and contains 740 hours, rather than a complete 744-hour January. Users should check time coverage and completeness instead of assuming every day has 24 records.

### Missing and invalid values

- **Fill value:** Missing or unavailable precipitation values are represented by IEEE **NaN** (`np.nan`) in the precipitation arrays.
- **Interpretation:** `NaN` indicates missing data and must **not** be interpreted as zero precipitation. A value of `0.0` indicates zero precipitation, whereas `NaN` indicates an unavailable estimate.
- **NetCDF encoding:** The missing-data convention is `NaN`; whether the final NetCDF files explicitly include an `_FillValue` attribute in addition to containing `NaN` values should be checked when finalizing CF metadata.
- **Processing:** Python/xarray recognizes floating-point `NaN` values as missing. When aggregating precipitation, check the number of valid hourly records rather than allowing `sum()` to silently skip missing hours.

## 4. Development, quality control, and validation

XTPrecip integrates complementary precipitation predictors from radar, satellite, reanalysis, and terrain information, with NOAA COOP hourly gauge observations serving as the supervised-learning target. CONUS-wide pretraining is followed by fine-tuning for seven NCA regions to better represent regional precipitation characteristics.

The modeling and assessment workflow includes screening gauge records, preparing temporally/spatially consistent predictor fields, evaluating withheld COOP test stations, and assessing precipitation characteristics such as frequency, intensity, and extremes. Existing comparison products include **NOAA Stage IV**, **AORC**, and the deep-learning-based **PUnet** precipitation dataset.

**Release-stage quality assurance is ongoing.** Final checks include missing/invalid values, implausible precipitation estimates, temporal coverage, regional-boundary artifacts, and precipitation-extreme behavior. Additional independent evaluation using **USCRN** observations is planned. Final verified quantitative performance results and any known limitations will be linked here when the associated manuscript and data release are finalized; no unverified accuracy improvement is claimed in this README.

## 5. Access and citation

### Data access

The public release is **planned by the end of 2026**, subject to final data checks and archive preparation. Data are **not yet available through AWS Open Data**.

When published, this section will contain:

- The official AWS Open Data Registry entry;
- The public S3 bucket name and `s3://` prefix;
- Instructions for browsing and downloading files (including whether anonymous access is enabled);
- A manifest or inventory of the monthly files; and
- Any recommended AWS-based workflows.

**AWS Registry URL:** [TO ADD AFTER PUBLICATION]  
**S3 URI:** [TO ADD AFTER PUBLICATION]  
**Repository URL:** [TO ADD AFTER PUBLICATION]

### License and recommended citation

**License:** Creative Commons Attribution 4.0 International (CC BY 4.0) is **proposed, pending approval** by the responsible institution and confirmation of upstream data-use terms. Do not treat this proposal as a finalized license until approval is documented.

A data descriptor manuscript is **planned for submission to *Earth System Science Data (ESSD)***. A recommended dataset citation, persistent identifier (DOI), version, publication date, and manuscript reference will be provided once available.

## 6. Example: Read, subset, and aggregate with Python/xarray

The following example opens one monthly NetCDF file, extracts a geographic subset, and calculates daily precipitation totals from the hourly `precip_scaled` variable. **Each timestamp marks the start of a one-hour accumulation interval**, and precipitation is expressed in mm h⁻¹. Daily aggregation by the date of the timestamp therefore sums hourly intervals beginning at 00:00 through 23:00 for that calendar day (provided all 24 values exist). The time standard (UTC or otherwise) still needs to be verified. Missing values are `np.nan` and are excluded from valid-hour counts.

Install the dependencies (with any suitable NetCDF backend):

```bash
python -m pip install xarray numpy netCDF4
```

```python
from pathlib import Path
import xarray as xr

# Update this path if the public archive has a different root directory.
filename = Path("blended_generation/2002/blended_precip_2002_01.nc")

with xr.open_dataset(filename, mask_and_scale=True, decode_times=True) as ds:
    print(ds)  # Inspect variables, dimensions, time, and attributes.

    # Prefer the final, intensity-scaled precipitation field (mm/h).
    hourly = ds["precip_scaled"]

    # Example bounding box: 32–36°N, 90–85°W.
    # Allow either ascending or descending coordinate order.
    def bounds_slice(coord, low, high):
        return slice(low, high) if coord.values[0] <= coord.values[-1] else slice(high, low)

    subset = hourly.sel(
        lat=bounds_slice(hourly["lat"], 32.0, 36.0),
        lon=bounds_slice(hourly["lon"], -90.0, -85.0),
    )

    # Each timestamp marks the START of its 1-hour accumulation interval.
    # Sum 1-hour rates to daily totals (mm) by interval start date.
    # Keep only grid-cell days with all 24 hourly values present.
    valid_hours = subset.resample(time="1D").count()
    daily_mm = subset.resample(time="1D").sum(min_count=1)
    daily_mm = daily_mm.where(valid_hours == 24)
    daily_mm.name = "daily_precipitation"
    daily_mm.attrs["units"] = "mm"
    daily_mm.attrs["long_name"] = "Daily total precipitation (complete days only)"

    # Simple spatial average over the selected grid cells (not area-weighted).
    regional_mean_daily_mm = daily_mm.mean(dim=("lat", "lon"), skipna=True)
    print(regional_mean_daily_mm)

    daily_mm.to_netcdf("ml4hp_us_daily_subset_2002_01.nc")
```

For multi-year or cloud-based analyses, use lazy loading/chunking (for example, Dask with xarray), and assess spatial weighting and time completeness according to the intended application.

## 7. Information to finalize before public release

- [ ] Verify the geographic coordinate attributes, geodetic datum, and CF metadata; no projected CRS is used.
- [ ] Verify UTC/calendar details in the published files (hour-start labeling is confirmed).
- [x] Document the missing-data fill value as `np.nan`.
- [ ] Check whether the final NetCDF files explicitly set `_FillValue` and confirm CF metadata consistency.
- [ ] Confirm archive structure, final variable names, compression, file sizes, and completeness.
- [ ] Complete final quality checks and add verified validation summaries and limitations.
- [ ] Approve the dataset license and document any source-data restrictions.
- [ ] Add the dataset DOI, citation, ESSD manuscript information, and official repository address.
- [ ] Add the AWS Registry/S3 access links and verify the published access example.

---

*This README is a pre-release draft for supervisory review and the AWS Open Data Sponsorship application. It should be publicly posted only after the metadata marked for confirmation have been reviewed, or with those limitations transparently retained.*
