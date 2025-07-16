# 📊 Data Profiling and Anomaly Detection

This document outlines techniques for **data profiling** and **anomaly detection** applied to datasets where approximately 10% of records are affected by zeros and NaNs. The goal is to enhance data quality before analysis or modeling.

<br>

## 🔍 Data Profiling Techniques

**Data profiling** helps understand the content, structure, and quality of your dataset.

### 1️⃣ Structure Discovery
- Analyze the format and consistency of your data.
- ✅ *Example:* Ensure all values in `Vmin_m3h` are numerical.

### 2️⃣ Data Types and Formats
- Verify correct data types for each column (numeric, text, date).
- ✅ *Tip:* Detect text values in numerical columns early.

### 3️⃣ Pattern Matching
- Check text fields for consistent patterns.
- ✅ *Example:* Device names should follow a format like `VAV-XXX`.

### 4️⃣ Content Discovery
#### 📈 Descriptive Statistics
- Compute min, max, mean, median, and standard deviation for numerical columns to spot potential outliers.

#### 📊 Value Frequency Analysis
- Count occurrences of each unique value in categorical columns.
- ✅ *Example:* Detect placeholder values like "Location" in `SetDeviceInstallationLocation`.

#### 🚨 Null Value Analysis
- Calculate the percentage of missing values per column to guide handling strategies.

### 5️⃣ Relationship Discovery
#### 🔑 Key and Dependency Analysis
- Identify potential primary keys (unique columns) and foreign keys (columns linking to other tables).
- ✅ *Example:* Check if `VmaxH_m3h` and `VmaxC_m3h` are always equal.

#### 🔗 Cross-Table Profiling
- Explore links between multiple tables for relational understanding.

<br>

## 🛠 Anomaly Detection and Error Removal Techniques

Given that ~10% of data is affected, cleaning should avoid bias. Here are methods for different anomaly types:

### 1️⃣ Handling Missing Values and Zeros
- ❌ Avoid simply deleting rows, as it may remove valuable information.

#### 🔄 Imputation
- **Mean/Median/Mode Imputation:** Replace missing numerical values with the mean or median, and categorical values with the mode.
- **Time-Series Specific Imputation:** Use techniques like Last Observation Carried Forward (LOCF) or Next Observation Carried Backward (NOCB).
- **Model-Based Imputation:** Apply k-Nearest Neighbors (k-NN) or regression models to predict missing values.
- **Flagging Missing Data:** Add a binary column to indicate originally missing values.

---

### 2️⃣ Inconsistent Categorical Data
- Standardize naming variations for consistency.

#### 🗂 Standardization
- Map common variations to a standard value.
- ✅ *Example:* Map `V8H_VAV_50E1_08` and `V8_VAV_5000_08` to `VAV_8`.

#### 📝 Rule-Based Cleaning
- Use scripts or regex to fix patterns or remove unwanted characters.

#### 🔍 Fuzzy Matching
- Group similar but non-identical strings using string similarity techniques.

<br>

### 3️⃣ Numerical Outliers
#### 📐 Z-Score Method
- Flag values with Z-scores > 3 or < -3 as potential outliers.

#### 🧠 Clustering-Based Methods
- Use algorithms like DBSCAN to detect anomalies outside of dense clusters.

<br>

### 4️⃣ Logical Inconsistencies
- Define validation rules to flag contradictions.

✅ **Examples:**
- `Vnom_m3h = 0` while `dp_at_Vnom_Cal ≠ 0` → Flag as **configuration error**.
- `Temp_Min > Temp_Max` → Clearly invalid, should be flagged.
- `Vmin_m3h > VmaxH_m3h or VmaxC_m3h` → Logical error needing correction.

<br>

### 5️⃣ Redundant Data
#### 🔗 Correlation Analysis
- Identify highly correlated numerical columns and consider removing redundant ones.

#### 👀 Manual Review
- For categorical columns like `device_name` and `objectName`, review a sample to check redundancy.

<br>

## ✅ Summary
Applying these data profiling and anomaly detection techniques will:
- Improve data quality.
- Prevent biases in analysis.
- Create a reliable foundation for analytics and modeling.
