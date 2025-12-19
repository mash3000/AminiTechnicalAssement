# Health Registry Data Cleaning Project

## Overview

This project profiles, cleans, and documents the National Health Facility Registry CSV dataset.

**Deliverables:**
- `health_registry_eda_clean.ipynb` - Jupyter notebook with EDA and cleaning pipeline
- `cleaned_health_registry.csv` - Cleaned dataset ready for analytics
- `health_registry_documentation.md` - Comprehensive documentation report
- `README.md` - This file
- `Loom video Link` - https://www.loom.com/share/6b2be447b24d46019b695b8a7ca0184b

---

## How to Run the Notebook

### Prerequisites

**Required Software:**
- Python 3.7 or higher
- Jupyter Notebook or JupyterLab

**Required Python Packages:**
```
pandas >= 1.3.0
numpy >= 1.20.0
```

### Installation

1. **Install Python** (if not already installed):
   - Download from [python.org](https://www.python.org/downloads/)
   - Or use a package manager: `brew install python` (Mac), `apt-get install python3` (Linux)

2. **Install Required Packages**:
   ```bash
   pip install pandas numpy jupyter
   ```

3. **Verify Installation**:
   ```bash
   python -c "import pandas, numpy; print('All packages installed successfully')"
   ```

### Execution Steps

1. **Start Jupyter Notebook**:
   ```bash
   jupyter notebook
   ```
   Or using JupyterLab:
   ```bash
   jupyter lab
   ```

2. **Open the Notebook**:
   - Navigate to the project directory
   - Open `health_registry_eda_clean.ipynb`

3. **Run All Cells**:
   - From the menu: `Cell` → `Run All`
   - Or use keyboard shortcut: `Shift + Enter` to run each cell sequentially

4. **Expected Output**:
   - EDA results and statistics printed in notebook cells
   - Cleaned dataset saved as `cleaned_health_registry.csv` in the project directory

5. **Verify Output**:
   ```bash
   # Check if cleaned file was created
   ls -lh cleaned_health_registry.csv
   
   # Or on Windows:
   dir cleaned_health_registry.csv
   ```

### Troubleshooting

- **Import Errors**: Ensure all packages are installed: `pip install pandas numpy`
- **File Not Found**: Ensure `health_registry.csv` is in the same directory as the notebook
- **Memory Issues**: The dataset is large (~13K rows). Close other applications if needed

---

## Key Assumptions/Decisions

### Cleaning Philosophy

**Balanced Approach**: The cleaning follows a balanced strategy:
- **Keep valid data** when there's reasonable certainty
- **Drop clearly invalid records** (test data, completely empty rows)
- **Document edge cases** and ambiguous situations
- **Preserve data** when uncertain rather than imputing

### Handling Ambiguous Cases

1. **Missing Facility IDs**: 
   - Records without IDs are retained if other information is valuable
   - Composite key (name + region) used for identification when ID is missing

2. **Date Parsing Failures**:
   - Invalid dates are set to NaN rather than dropping the entire record
   - Date anomalies (inspection before licence) are flagged but preserved

3. **GPS Coordinate Ambiguity**:
   - Coordinates outside Barbados bounds are set to NaN (not dropped)
   - Ambiguous lat/lon order resolved using geographic bounds logic

4. **Near-Duplicates**:
   - Facilities with same name but different IDs kept if in different regions
   - Same name + same region: keep most complete record

### Duplicate Resolution Strategy

1. **Exact Duplicates**: Removed entirely (keeping first occurrence)
2. **Near-Duplicates**: 
   - Identified by normalized name + region
   - Completeness scoring: Counts non-null values across key fields
   - Kept record with highest completeness score

**Completeness Score Components:**
- Facility name
- Facility type
- Capacity
- Licence date
- Inspection date
- GPS coordinates
- Remarks

### Date Parsing Assumptions

**Format Priority Order:**
1. Standard formats (YYYY-MM-DD, DD-MM-YY, etc.)
2. Explicit format strings tried in sequence
3. Pandas flexible parsing as fallback

**Assumptions:**
- 2-digit years assumed to be 2000s (e.g., 21 → 2021)
- Ambiguous dates (e.g., 01-02-03) parsed using format priority
- Invalid dates (parsing failures) set to NaN

### Region Normalization Rules

**Canonical Names:**
- Standardized to: "St. James", "St. Lucy", "St. Peter", "St. Andrew", "St. Michael", "St. Joseph", "St. John", "St. George", "Christ Church"

**Normalization Steps:**
1. Case normalization (Title Case)
2. Mapping of variants (e.g., "ST. JAMES" → "St. James")
3. Handling reversed text from OCR errors (e.g., "semaJ .tS" → "St. James")
4. Standardizing "Parish" suffix usage

**Note**: Some edge cases may not be fully normalized (e.g., some reversed patterns may remain). See documentation for details.

### Capacity Parsing Rules

**Text Patterns Handled:**
- "ten beds" → 10, "beds"
- "250 beds" → 250, "beds"
- "N/A", "unknown" → NaN, NaN
- Numeric extraction from mixed text

**Units Standardized To:**
- beds
- cots
- patients
- capacity
- units (default fallback)

**Assumptions:**
- Capacity is always a positive integer
- Units are preserved separately from numeric value
- Missing capacity values not imputed

### GPS Coordinate Processing

**Formats Supported:**
- POINT(lon lat)
- Comma-separated: "lat, lon" or "lon, lat"
- Semicolon-separated: "lon;lat" or "lat;lon"
- Degree-minute-second: 13°12′43″N 58°51′50″W

**Validation:**
- Coordinates validated against Barbados geographic bounds:
  - Latitude: 12.5 to 13.5
  - Longitude: -59.7 to -58.5
- Out-of-range coordinates set to NaN (not dropped)

**Order Resolution:**
- POINT format: Explicit order (lon lat)
- Other formats: Resolved using value ranges and geographic bounds

---

## File Descriptions

| File | Description |
|------|-------------|
| `health_registry.csv` | **Input**: Raw dataset (13,366 rows, 8 columns) |
| `health_registry_eda_clean.ipynb` | **Notebook**: EDA and cleaning pipeline |
| `cleaned_health_registry.csv` | **Output**: Cleaned dataset (10,318 rows, 12 columns) |
| `health_registry_documentation.md` | **Documentation**: Comprehensive profiling and findings report |
| `README.md` | **Guide**: This file |

---

## Data Dictionary

### Cleaned Dataset Columns

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| **facility_id** | integer | Unique facility identifier | Numeric portion extracted from original ID (e.g., "HF-0001" → 1, "033477" → 33477) |
| **facility_name** | string | Name of health facility | "Burgess-Ingram Medical Center" |
| **facility_type** | string | Type of facility | "Hospital", "Health Centre", "Clinic" |
| **capacity_numeric** | float | Numeric capacity value | 250.0, 117.0, 10.0 |
| **capacity_unit** | string | Unit of capacity | "beds", "cots", "patients", "capacity" |
| **region** | string | Geographic region/parish | "St. James", "Christ Church" |
| **licence_issue_date** | datetime | Date licence was issued | 2021-04-01 |
| **inspection_date** | datetime | Date of last inspection | 2024-03-08 |
| **latitude** | float | Geographic latitude (decimal degrees) | 13.08576 |
| **longitude** | float | Geographic longitude (decimal degrees) | -58.75331 |
| **remarks** | string | Additional notes/status | "Good", "Follow-up 2023", "Reinspection due" |
| **date_anomaly** | boolean | Flag for date logic issues | True, False |

### Column Notes

- **facility_id**: Integer type (numeric portion only extracted). May be missing (~28.5% of records). Use facility_name + region as composite key when ID is missing.
- **facility_type**: Standardized categories. May be missing (~15.2%).
- **capacity_numeric**: Missing for ~31.2% of records. Always check capacity_unit when capacity_numeric is present.
- **region**: Nearly complete (>99%). Standardized to canonical names.
- **dates**: Parsed to datetime objects. Missing for ~1% of records.
- **latitude/longitude**: Missing for ~17.3% of records. Validated to Barbados bounds.
- **date_anomaly**: True if inspection_date < licence_issue_date (may indicate data quality issue).

---

## Quick Start

**For Quick Analysis:**
```python
import pandas as pd

# Load cleaned data
df = pd.read_csv('cleaned_health_registry.csv', parse_dates=['licence_issue_date', 'inspection_date'])

# Basic exploration
print(df.shape)
print(df.info())
print(df.describe())

# Filter facilities with GPS coordinates
df_with_coords = df[df['latitude'].notna()]
print(f"Facilities with coordinates: {len(df_with_coords)}")
```

**For Geographic Analysis:**
```python
# Filter valid coordinates
df_geo = df[(df['latitude'].notna()) & (df['longitude'].notna())]

# Count by region
print(df_geo.groupby('region').size())
```

**For Capacity Analysis:**
```python
# Facilities with capacity data
df_capacity = df[df['capacity_numeric'].notna()]

# Summary statistics
print(df_capacity.groupby(['facility_type', 'capacity_unit'])['capacity_numeric'].describe())
```

---

## Questions or Issues

For questions about:
- **Data cleaning logic**: See `health_registry_eda_clean.ipynb`
- **Profiling results**: See `health_registry_documentation.md`
- **Data quality issues**: Check the "Major Findings" section in documentation

---

## License

This project processes publicly available health facility registry data. Ensure compliance with data privacy regulations when using this dataset.




