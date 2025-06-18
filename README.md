# Urban Wind Modeling with WRF

![WRF Validation Example Plot](/wrf_vs_observed.png)  
*Example comparison of WRF predictions vs observed wind speeds*

## Project Description
Validation system for evaluating Weather Research and Forecasting (WRF) model performance at different:
- **Resolutions**: 1m, 2m, 5m
- **Urban Physics Schemes**:
  - `0` = Control (Basic WRF)
  - `1` = SLUCM (Single Layer Urban Canopy Model)
  - `2` = BEP (Building Effect Parameterization)
  - `3` = BEP+BEM (Building Energy Model)

## Results Summary
### Prediction Accuracy
Insert your actual metrics table here (replace the example):

| Resolution | Scheme   | MAE (m/s) | RMSE (m/s) |
|------------|----------|-----------|------------|
| 1m         | BEP+BEM  | 1.12      | 1.98       |
| 2m         | BEP      | 1.45      | 2.31       |
| 5m         | Control  | 2.01      | 2.87       |

### Computational Performance
![Runtime Comparison](plots/runtime_comparison.png)  
*Actual runtime data across configurations*

## Implementation
### Core Code Structure
```python
# Configuration
resolutions = ["1m","2m","5m"]
usf_configs = {0:"Control", 1:"SLUCM", 2:"BEP", 3:"BEP+BEM"}

# Main validation workflow
for res in resolutions:
    for usf, scheme in usf_configs.items():
        # 1. Load WRF outputs
        wrf_files = sorted(glob(f'data/{res}_uf{usf}/wrfout*'))
        wrf_data = [Dataset(f) for f in wrf_files]
        
        # 2. Extract wind speed at 10m height
        wspd10 = getvar(wrf_data, "wspd", units="m/s")
        
        # 3. Map predictions to observation points
        obs_df['WRF_WIND'] = obs_df.apply(
            lambda row: get_wrf_wind_at_time_and_location(
                row['LATITUDE'],
                row['LONGITUDE'],
                row['DATE']
            ),
            axis=1
        )
        
        # 4. Calculate metrics
        mae = np.mean(np.abs(obs_df['WND_SPEED'] - obs_df['WRF_WIND']))
        rmse = np.sqrt(mean_squared_error(obs_df['WND_SPEED'], obs_df['WRF_WIND']))
        
        # 5. Generate visualization
        generate_validation_plot(obs_df, res, scheme, mae, rmse)
