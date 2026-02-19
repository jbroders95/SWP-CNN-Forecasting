# SWP-CNN-Forecasting

## Overview
This repository contains the convolutional neural network (CNN) forecasting framework used to predict short-term shoaling accumulation trends in the Southwest Pass of the Mississippi River as described in:

**Broders et al. (2026), _Computers & Geosciences_**

The model is trained on gridded hydrographic bathymetric survey data (eHydro) and predicts a future shoaling proxy defined as the percentage of channel area below dredging threshold depth. Forecasts are generated using spatial CNNs applied to downscaled bathymetric raster time-series derived from USACE survey data.

---

## Method Summary

### Input
Bathymetric survey rasters are extracted from reach-level Zarr datasets:

SW_0{reach}_SWP_01_YYYY-MM-DD.zarr

Surfaces are:
- Downscaled using spatial averaging  
- Normalized between minimum and maximum channel depths  
- Masked where survey coverage is unavailable  

Temporal sequences are constructed using:
- **8 weeks input** (weeks_in = 8)  
- **4 weeks forecast horizon** (weeks_out = 4)  

Model input tensor shape:

(batch, height, width, 8)

Each channel corresponds to one weekly bathymetric surface.

---

### Target Variable
Shoaling state is quantified as:

**Percent of channel grid cells below dredging threshold depth**

Threshold values:

| Date Range        | Threshold Depth |
|-------------------|-----------------|
| Before 2020-01-01 | 45 ft           |
| After 2020-01-01  | 50 ft           |

This scalar target is min-max scaled prior to training.

---

### Model Architecture
The forecasting model consists of:
- 2D convolutional layers  
- Spatial max pooling  
- Batch normalization  
- Global average pooling  
- Dropout regularization  
- Dense regression output  

**Loss Function:** Mean Squared Error (MSE)  
**Optimizer:** Adam (learning rate = 5e-5)

Monte Carlo Dropout is used during inference to estimate predictive uncertainty.

---

## Installation

Clone the repository:

```bash
git clone https://github.com/YOUR_USERNAME/SWP-CNN-Forecasting.git
cd SWP-CNN-Forecasting
```

Create the environment:

```bash
conda env create -f environment.yml
conda activate swp-cnn
```

---

## Data Requirements
Hydrographic surveys used in this work are available from:

- USACE eHydro  
  https://www.arcgis.com/apps/dashboards/4b8f2ba307684cf597617bf1b6d2f85d

Raw survey data must be converted into:

Zarr raster time-series format

with expected variable:

depth(time, y, x)

---

## Training

Run:

SWP_CNN_forecasting.ipynb

Model weights will be saved as:

reach_<reach>_CNN.keras

---

## Forecast Inference
Shoaling forecasts with uncertainty estimation are generated using Monte Carlo Dropout:

```python
monte_carlo_dropout(model, input_data, num_samples=100)
```

Outputs:
- Mean predicted shoaling state  
- Predictive uncertainty (standard deviation)

---

## Reproducing 4-Week Shoaling Forecast

Load trained weights:

reach_<reach>_CNN.keras

Construct validation input sequences:

```python
validation_X = sequence_depths_X[-6:-5]
```

Run inference:

```python
mean_pred, uncertainty = monte_carlo_dropout(model, validation_X)
```

---

## License
MIT License

---

## Citation

If you use this repository, please cite:

```bibtex
@software{SWP_CNN_2026,
  author  = {Broders, Hans},
  title   = {SWP-CNN-Forecasting},
  year    = {2026},
  version = {1.0},
  url     = {https://github.com/YOUR_USERNAME/SWP-CNN-Forecasting}
}
```

A DOI will be assigned upon archival via Zenodo.
