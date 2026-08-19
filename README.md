# README: `final_report.ipynb`

## Project title

**Spatial Downscaling of SMAP Soil Moisture Using Multi-Modal Machine Learning**

## Purpose of the notebook

`final_report.ipynb` contains the final machine-learning workflow for a capstone project on soil moisture downscaling in southwestern Oklahoma. The notebook trains regression models to learn the relationship between coarse SMAP soil moisture and higher-resolution proxy variables, then applies the selected model to produce a 1 km downscaled soil moisture product for April–September 2024.

The notebook also includes diagnostic analysis, model tuning, residual evaluation, 1 km prediction mapping, and preliminary validation against an ISMN in-situ soil moisture station.

## Problem summary

SMAP soil moisture is useful for regional environmental monitoring, but its spatial resolution is too coarse for many local applications. This project treats downscaling as a supervised regression problem: the model learns from SMAP-scale soil moisture and aggregated proxy variables, then applies the learned relationship to 1 km predictor data.

The main target variable is:

- `smap_sm`: SMAP Level 3 Enhanced near-surface soil moisture

The main predictor variables are:

- Sentinel-1 radar: `vv`, `vh`, `angle`, `vv_minus_vh`
- MODIS NDVI: `ndvi`
- SRTM terrain variables: `elevation`, `slope`, `aspect_sin`, `aspect_cos`
- Temporal feature: `month`

## Study area and time period

- **Region:** Southwestern Oklahoma, near the Little Washita / Fort Cobb / Fort Reno SMAP validation region
- **ROI:** longitude -99.0 to -97.7 and latitude 34.5 to 35.8
- **Time period:** April 1, 2024 through September 30, 2024
- **Prediction resolution:** 1 km

## Notebook structure

The notebook is organized into the following major sections:

1. **Random Forest diagnosis**  
   Reproduces the initial temporal split results and investigates why the original Random Forest predictions were compressed when trained on April–August and tested on September.

2. **Temporal generalization diagnostics**  
   Compares Random Forest models with and without `month`, including random split and leave-one-month-out testing.

3. **Final model tuning**  
   Uses a stratified random split by month and tunes Lasso Regression and Random Forest models with and without `month`.

4. **Final model selection and residual analysis**  
   Selects the tuned Random Forest with `month` as the best in-distribution model and checks residuals by month and location.

5. **1 km downscaling**  
   Retrains the selected model on all SMAP-scale samples and applies it to the 1 km prediction table.

6. **ISMN validation**  
   Processes the Fort Reno ISMN soil moisture data, calculates monthly station averages, matches them to the nearest 1 km prediction, and calculates validation metrics.

## Required input files

The notebook expects the following files to be available in the working directory or in the `data/` subfolder:

| File | Purpose |
|---|---|
| `data/oklahoma_smap_scale_training_Apr_Sep_2024.csv` | SMAP-scale training table exported from Google Earth Engine |
| `data/oklahoma_1km_prediction_Apr_Sep_2024.csv` | 1 km predictor table exported from Google Earth Engine |
| `data/Data_separate_files_header_20240401_20240930_9066_ICHg_20260815.zip` | ISMN download containing Fort Reno `.stm` soil moisture files |

The notebook path variables can be edited if the files are saved somewhere else.

## Required Python packages

The notebook uses the following main Python packages:

```bash
pip install pandas numpy matplotlib scikit-learn ismn
```

If using a conda environment, activate the environment first, for example:

```bash
conda activate mlcourse2
pip install pandas numpy matplotlib scikit-learn ismn
```

The notebook was written for a Jupyter environment such as VS Code Jupyter or Jupyter Notebook.

## How to run the notebook

1. Place the required CSV and ISMN ZIP files in the expected folder structure.
2. Open `final_report.ipynb` in Jupyter or VS Code.
3. Confirm the file paths in the data-loading cells:
   - Cell 2 for the SMAP-scale training data
   - Cell 34 for the 1 km prediction table
   - Cell 39 for the ISMN ZIP file
4. Run the notebook from top to bottom.
5. Check the generated plots, tables, saved CSV files, and validation metrics.

## Main models evaluated

The notebook compares the following supervised regression models:

- Mean baseline model
- Lasso Regression with `month`
- Lasso Regression without `month`
- Random Forest Regression with `month`
- Random Forest Regression without `month`

The final selected model is:

**Tuned Random Forest Regression with `month`**

This model was selected because it had the strongest performance under the final stratified random split by month.

## Key final model results

Under the final stratified random split, the tuned Random Forest with `month` produced the best test-set performance:

| Model | RMSE | MAE | R² |
|---|---:|---:|---:|
| Tuned Random Forest with `month` | 0.0094 | 0.0067 | 0.9545 |
| Tuned Random Forest without `month` | 0.0232 | 0.0182 | 0.7192 |
| Tuned Lasso with `month` | 0.0231 | 0.0187 | 0.7232 |
| Tuned Lasso without `month` | 0.0274 | 0.0224 | 0.6099 |
| Mean baseline | 0.0439 | 0.0362 | -0.0002 |

The results show that the tuned Random Forest with `month` performs best for in-distribution prediction within the April–September 2024 study period.

## Important diagnostic finding

The `month` variable improves in-distribution performance, but it can also act as a seasonal shortcut. Leave-one-month-out diagnostics showed that temporal generalization is harder when the model is asked to predict a month excluded from training. Therefore, the selected model is best interpreted as an April–September 2024 in-distribution downscaling model, not as a fully general future-year forecasting model.

For future-year prediction or unusual seasonal conditions, the Random Forest without `month` may be more physically defensible, especially if precipitation, evapotranspiration, or antecedent moisture variables are added.

## Downscaled output

The final model is retrained on all SMAP-scale samples using the selected hyperparameters and then applied to the 1 km prediction table. This creates the final output column:

- `rf_downscaled_sm`: Random Forest 1 km downscaled soil moisture prediction

The saved downscaled prediction file is:

```text
oklahoma_1km_downscaled_rf_with_month_Apr_Sep_2024.csv
```

The `smap_sm` column in the 1 km table is treated only as a coarse SMAP reference sampled at 1 km locations. It is not true 1 km ground-truth soil moisture.

## ISMN validation summary

The notebook performs preliminary external validation using the Fort Reno ISMN station at approximately 5 cm depth. Validation is based on six monthly station observations for April–September 2024.

RF downscaled validation against ISMN:

| Metric | Value |
|---|---:|
| Number of station-months | 6 |
| RMSE | 0.0504 |
| MAE | 0.0428 |
| Bias | -0.0416 |
| Correlation | 0.9282 |

The high correlation suggests that the downscaled product captures the general month-to-month pattern. However, the negative bias shows that the RF predictions are systematically lower than the Fort Reno in-situ observations. This is expected because the model was trained to reproduce SMAP-scale soil moisture, not point-scale ground measurements.

The ISMN validation should be interpreted as preliminary because it uses only one station and six monthly observations.

## Output files generated by the notebook

| Output file | Description |
|---|---|
| `selected_random_forest_feature_importance.csv` | Feature importance table for the selected tuned Random Forest model |
| `selected_random_forest_test_residuals.csv` | Test-set residuals for the selected model |
| `selected_random_forest_error_summary_by_month.csv` | Monthly error summary for the selected model |
| `oklahoma_1km_downscaled_rf_with_month_Apr_Sep_2024.csv` | Final 1 km downscaled soil moisture table |
| `ismn_monthly_soil_moisture_004_006m_Apr_Sep_2024.csv` | Monthly Fort Reno ISMN validation table |
| `ismn_validation_nearest_1km_rf_downscaled_Apr_Sep_2024.csv` | Nearest-pixel ISMN validation comparison table |
| `ismn_validation_summary_rf_downscaled_Apr_Sep_2024.csv` | Summary metrics for ISMN validation |

## Limitations

The final results should be interpreted with the following limitations:

- The model is trained on SMAP-scale soil moisture, so the downscaled product is SMAP-consistent rather than a direct independent 1 km soil moisture retrieval.
- ISMN validation is limited to one station and six monthly observations.
- Point-scale ISMN measurements and 1 km satellite-derived predictions are not perfectly comparable because of spatial scale mismatch.
- The `month` feature improves final in-distribution performance but may reduce transferability to future years or abnormal seasonal conditions.
- Additional meteorological predictors such as precipitation, temperature, evapotranspiration, and antecedent soil moisture would likely improve future versions.

## Recommended next steps

Future improvements could include:

1. Expand the validation period to multiple years.
2. Add more ISMN stations by expanding the ROI or search buffer.
3. Add meteorological variables such as precipitation and evapotranspiration.
4. Compare the RF with `month` model against a no-month model for future-year transfer testing.
5. Try anomaly-based downscaling to reduce seasonal shortcut behavior.
6. Compare the downscaled product against existing SMAP downscaling products or other independent soil moisture datasets.

## Notes for reviewers

The written final report is designed to summarize the problem, methods, results, findings, and next steps for a non-technical reader. This notebook contains the detailed coding workflow, diagnostic checks, model tuning, plots, and validation calculations used to support the report.
