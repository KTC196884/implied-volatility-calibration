# Implied Volatility Calibration

A small Python package plus helper scripts that:

* clean and resample Taiwan option / futures data  
* compute Black–Scholes **implied volatilities (IV)**  
* calibrate the **SVI** volatility surface  
* create interactive HTML plots

## Project Structure

```text
implied-volatility-calibration/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/
│   ├── raw/
│   ├── interim/
│   └── final/
│
├── src/
│   └── iv_calibration/  # pip-installable package
│       ├── __init__.py
│       ├── config.py
│       ├── data_preprocessor.py
│       ├── svi_calibrator.py 
│       └── visualization/
│           └── svi_plotter.py
│
├── scripts/
│   ├── run_data_preprocessor.py
│   ├── run_svi_calibrator.py
│   └── run_svi_plotter.py
│
├── tests/
│   ├── test_data_preprocessor.py
│   ├── test_svi_calibrator.py
│   └── test_svi_plotter.py
│
└── results/
```
---

### requirements.txt

```text
numpy>=1.21.0,<2.0
pandas>=1.3.0,<2.0
scipy>=1.7.0,<2.0
matplotlib>=3.4.0,<4.0
plotly>=5.0.0,<6.0
```
---

## Installation

```bash
# 1. create & activate virtual env
python3 -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate

# 2. install dependencies
pip install -r requirements.txt 
```

## Usage

### Import as a Python package
```python
from pathlib import Path
import pandas as pd

# Package components
from iv_calibration import data_preprocessing as dp
from iv_calibration import compute_svi_params, PATHS
from iv_calibration.visualization import svi_plotter as sp

# ----------------------------------------------------------------
# STEP 1 Clean raw data, compute IV, and write parquet
# ----------------------------------------------------------------
dp.main()                                        # writes data/interim & data/final

# ----------------------------------------------------------------
# STEP 2 Fit SVI surface
# ----------------------------------------------------------------
option_resampled_df = pd.read_parquet(PATHS.option_resampled)
params_df           = compute_svi_params(option_resampled_df)
params_df.to_parquet(PATHS.vol_surface_svi)

# ----------------------------------------------------------------
# STEP 3 Interactive plots
# ----------------------------------------------------------------
sp.plot_with_slider(
    option_resampled_df,
    params_df,
    sp.build_svi_iv_curve,
    "iv",
    "Implied Volatility",
    Path("results/svi_iv_slider.html")
)
```
dp.main() and the plotting helpers are convenience wrappers;
for fine‑grained control import the underlying functions directly.

### Command-line scripts
```bash
# Pre‑process raw CSVs  →  parquet
python scripts/run_data_preprocessor.py \
       --input  data/raw \
       --output data/final

# Calibrate SVI  →  params parquet
python scripts/run_svi_calibrator.py \
       --input  data/final/option_resampled.parquet \
       --output results

# Generate interactive HTML plots
python scripts/run_svi_plotter.py \
       --input  results \
       --output results
```

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.