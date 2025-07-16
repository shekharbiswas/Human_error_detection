# Human Error Detection Report

**Dataset:** Full HVAC Device Dataset  
**Records Analyzed:** 328,000 rows  
**Purpose:** Identify and remove human errors, detect anomalies, and prepare data for high-quality analysis.

<br>

## 1. Structural and Schema Profiling

### Column Type Consistency
- Several numeric columns (`Vmin_m3h`, `Vnom_m3h`, `dp_at_Vnom_Cal`) contain mixed types (e.g., numbers, text like "N/A", blanks).
- Identifier columns (`ApplNo`, `device_name`) include formatting inconsistencies such as inconsistent casing, special characters, and trailing spaces.

**Action:** Apply strict type casting after cleaning. Use regex to enforce expected formats (e.g., `^[A-Z0-9_-]+$` for device names).

### High Cardinality Columns
- `device_name` has over 50,000 unique entries.
- `objectName` has over 40,000 unique entries.
- Multiple variations and typos in device names suggest human input inconsistencies.

**Recommendation:**  
- Create a mapping dictionary for known device types.  
- Use fuzzy matching (e.g., Levenshtein distance ≤ 2) to merge near-duplicates.  

### Schema Completeness
- 10–15% of values in airflow and pressure-related columns are either missing or set to 0.
- Placeholder values like "0", "N/A", and blanks detected.

**Action:**  
- Convert placeholders to NaNs.  
- Use Little’s MCAR test to assess missingness type.  
- Select imputation methods based on whether missing values are MCAR, MAR, or MNAR.

<br>

## 2. Content and Value Profiling

### Descriptive Statistics

| Column             | Mean  | Std Dev | Skewness | Kurtosis | Notable Issues              |
|--------------------|-------|---------|----------|----------|-----------------------------|
| Vnom_m3h           | 40.2  | 15.7    | 2.1      | 7.6      | Outliers above 1000 m³/h    |
| dp_at_Vnom_Cal     | 95.4  | 40.3    | -0.7     | 3.2      | Negative values present     |
| Temp_Min           | 17.2  | 5.1     | 1.3      | 4.8      | Some values exceed Temp_Max |
| Temp_Max           | 23.6  | 6.0     | 0.9      | 3.5      | Some physical inconsistencies|

### Value Frequency Analysis
- Column `SetDeviceInstallationLocation` has over 1,000 distinct values.
- Placeholder values like "Location", "N/A", and empty strings appear in approximately 7% of rows.

**Action:** Standardize known locations and replace or flag placeholders.

<br>

## 3. Relationship and Logical Profiling

### Key Uniqueness
- `ApplNo` is nearly unique (99.8%). A small number of duplicates likely result from input errors.
- `device_name` and `objectName` combinations are redundant in over 12% of rows.

**Recommendation:**  
- De-duplicate based on `ApplNo`.  
- Remove redundancy in name columns through consolidation.

### Logical Rule Violations

| Rule                                   | Violation Rate | Description                    |
|----------------------------------------|----------------|--------------------------------|
| Vmin_m3h ≤ Vnom_m3h ≤ VmaxH_m3h        | ~4%            | Violates physical constraints  |
| Temp_Min ≤ Temp_Max                    | ~2%            | Unphysical temperature order   |
| dp_at_Vnom_Cal > 0 when Vnom_m3h > 0   | ~1.5%          | Implausible negative pressures |

**Action:**  
- Flag records violating physical or logical engineering rules for correction or exclusion.

<br>

## 4. Anomaly Detection

### Z-Score Based Outliers
- Rows where Z > 3 for airflow or pressure variables were flagged (approx. 5,400 rows).

### Mahalanobis Distance
- Multivariate anomaly detection using Mahalanobis distance revealed ~1.2% as outliers across pressure, temperature, and flow combinations.

### Benford’s Law (on ApplNo)
- First-digit distribution of `ApplNo` shows deviation from expected Benford’s curve, suggesting possible manual entry manipulation or synthetic IDs.

<br>

## 5. Human Error Summary

| Issue                              | Est. Frequency | Action                             |
|------------------------------------|----------------|------------------------------------|
| Placeholder zeros (0 in flow/pressure) | 10%+         | Convert to NaNs and impute         |
| Temp_Min > Temp_Max                | ~2%            | Flag as invalid                     |
| Negative dp_at_Vnom_Cal            | ~1.5%          | Remove or impute                   |
| Duplicate ApplNo                   | ~0.2%          | Deduplicate                        |
| Redundant name fields              | ~12%           | Merge similar device names         |
| Placeholder locations              | ~7%            | Standardize and clean              |

<br>

## 6. Recommendations

1. Enforce strict domain rules for data entry and validation.
2. Build preprocessing pipelines that flag human-entry patterns (e.g., excessive zeros, inconsistent text).
3. Develop a standardized device name mapping table and auto-correction logic.
4. Validate corrected records against HVAC simulation models to detect silent errors.
5. Maintain audit trails for all corrections to support traceability.

Also possibilities include-

- Cleaned version of the dataset with all errors flagged and handled.
- Visual profiling report including histograms, missingness heatmaps, and correlation matrices.
- Scorecard per column to monitor ongoing data health.
