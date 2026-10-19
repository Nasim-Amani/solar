# Solar & PV Measurement Data: Cleaning and Exploratory Analysis

Outdoor solar and PV measurements at **1-second resolution**, 15–27 February 2013 (1,015,991 records; 11 complete days, Feb 16–26).

## Roadmap

```
17 Excel files (.xlsx)
        │  0cleaned_solar_data_Oct26.ipynb
        ▼
cleaned_solar_data_Oct26.csv
        │  1solar_data_SolarRad_clean.ipynb
        ▼
solar_data_SolarRad_clean.csv
        │  2solar_data_final_columns.ipynb
        ▼
solar_data_final_columns.csv
        │  3Exploratory_Analysis_Correlation_and_Thermal_Response.ipynb
        ▼
Exploratory analysis & thermal response (no new file)
```

### 0 · Merging and basic cleaning: `0cleaned_solar_data_Oct26.ipynb`
- Read and merged **17 Excel files** (60,000 rows each, identical structure).
- Built a single `DateTime` from `Date` + `Time`.
- Removed **17 exact duplicate records** and corrected repeated timestamps so that records are strictly 1 s apart (originals kept during the check).
- Checked missing values, zeros and temperature ranges: no missing values or impossible values.
- Dropped empty or unused columns (`Parameter 8`, `PV 2 GT V`, `PV 3 Tr V`) and helper columns, and renamed the remaining columns.
- **Output:** `cleaned_solar_data_Oct26.csv`

### 1 · Solar radiation cleaning: `1solar_data_SolarRad_clean.ipynb`
- Checked timestamps: no duplicates, 9 isolated missing seconds, **11 complete days** → `full_day` flag.
- Inspected the daily cycle: clear and cloudy days, and a constant night reading of ~2.7 W/m².
- **Removed the sensor offset** (night median 2.7134 W/m²) from all values and set negatives to 0 → `Solar_Rad_clean`.
- Defined daylight as a 1-min median of `Solar_Rad_clean` > 1 W/m² → `daylight` flag.
- **Output:** `solar_data_SolarRad_clean.csv`

### 2 · Column screening: `2solar_data_final_columns.ipynb`
- Profiled every sensor (range, day/night behaviour, correlation with irradiance).
- **`PV_1_SA_V`** confirmed as **open-circuit voltage (Voc)** (temperature coefficient −0.325 %/°C).
- **`PV_4`** identified as a small panel **charging a battery through a charge controller**; added power `PV_4_Hor_P` and state `PV4_state`.
- Removed unusable columns: `PV_1_SA_I` (open circuit, ~0 A) and `PV_3_Tr_I` (not working consistently).
- **Output:** `solar_data_final_columns.csv`

### 3 · Exploratory analysis: `3Exploratory_Analysis_Correlation_and_Thermal_Response.ipynb`
- Descriptive statistics (day vs night) and daily behaviour of all variables.
- Correlation analysis: **Pearson, Spearman** and **mutual information** (on 1-min changes).
- **Thermal response:** module temperatures react to irradiance changes within ~1 min, with thermal time constants of **~5–12 min**.

## Final dataset: `solar_data_final_columns.csv`

| Column | Meaning |
|---|---|
| `Solar_Rad_clean` | Solar irradiance, offset-corrected (W/m²) |
| `Solar_Rad` | Raw irradiance (reference only) |
| `Amb_Tempr` | Ambient temperature (°C) |
| `Tempr_PV_1`, `Tempr_PV_2`, `Tempr_PV_3` | PV module temperatures (°C) |
| `PV_1_SA_V` | Open-circuit voltage of PV module 1 (V) |
| `PV_2_GT_I` | Current of PV module 2 (A) |
| `PV_4_Hor_V`, `PV_4_Hor_I`, `PV_4_Hor_P`, `PV4_state` | Battery-charging panel: voltage, current, power, operating state |
| `full_day`, `daylight` | Flags for complete days and daylight records |
