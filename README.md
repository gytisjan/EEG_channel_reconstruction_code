# EEG Missing Channel Reconstruction Using Interpolation Methods

A comparative study of spatial interpolation methods for reconstructing a missing electroencephalography (EEG) channel. The project reconstructs **F3** using four neighboring electrodes (**Fp1, F7, C3, and Fz**) and compares the reconstructed signal against the original recording.

This repository contains the Python/Jupyter notebooks used for EEG preprocessing, channel reconstruction, and comparative analysis.

## Methods

| Notebook | Purpose |
| --- | --- |
| `Data_cleaning.ipynb` | Preprocess the EEG recordings and save cleaned FIF files. |
| `Linear_interpolation.ipynb` | Reconstruct F3 using two-dimensional linear spatial interpolation (`LinearNDInterpolator`). |
| `Cubic_spline.ipynb` | Reconstruct F3 using two-dimensional Clough–Tocher piecewise cubic interpolation. |
| `IDW.ipynb` | Reconstruct F3 using inverse distance weighting (IDW) with three-dimensional electrode distances and power `p = 2`. |
| `Spherical_spline.ipynb` | Reconstruct F3 using spherical spline interpolation based on electrode positions projected onto a unit sphere. |
| `Analysis.ipynb` | Compare saved results and visualize PSD, coherence, amplitude histograms, and log–log spectral slopes. |

The recorded F3 channel is **not used as an input to the reconstruction**. It is retained as a reference for evaluating each method.

## Data and preprocessing

The notebooks process **29 eyes-closed EEG recordings**, with identifiers `sub-037` through `sub-065`, stored in EEGLAB `.set` format. The preprocessing notebook:

- removes boundary annotations without removing EEG samples;
- applies a **1–45 Hz** band-pass filter;
- sets an average EEG reference;
- fits a FastICA model and saves it along with the cleaned recording.

**ICA components are not removed by default** (`auto_remove_ica_components = False`). Thus, fitting ICA does not itself mean that artifact components were rejected from the exported cleaned signal.

The interpolation notebooks use electrode positions from `coordinates.csv`. The **Fz position was absent from the original coordinate file and was calculated separately** during the study; its fixed coordinates are included in the interpolation notebooks.

> **Data provenance to complete before making the repository public:** add the original EEG dataset citation/download page, the source of `coordinates.csv`, the method used to obtain Fz coordinates, and the applicable reuse or redistribution conditions. These details cannot be established from the code alone.

The original recordings and generated signal files are intentionally excluded from this repository. Please obtain EEG recordings from the original authorized source under its terms of use.

## Project structure

```text
EEG_channel_reconstruction_code/
├── Data_cleaning.ipynb
├── Linear_interpolation.ipynb
├── Cubic_spline.ipynb
├── IDW.ipynb
├── Spherical_spline.ipynb
├── Analysis.ipynb
├── coordinates.csv                 # Include only if redistribution is allowed
├── README.md
├── requirements.txt
├── .gitignore
├── Raw_data/                      # Local only; not tracked by Git
│   └── sub-037_task-eyesclosed_eeg.set
├── Processed_data/                # Generated locally; not tracked by Git
│   ├── sub-037_task-eyesclosed_eeg_cleaned_raw.fif
│   ├── F3_linear_spatial_interpolation_metrics_all_subjects.csv
│   └── Reconstructed_signals/
└── Analysis_results/              # Generated locally; not tracked by Git
```

The data filenames in the tree are examples. The pipeline expects the filenames and subject numbering used by the notebooks. Some EEGLAB `.set` files may also require an accompanying `.fdt` file.

## Installation

**Python 3.12** is a suitable starting point for the notebooks. From the project root, create a virtual environment and install the dependencies:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m jupyter lab
```

Alternatively, activate your existing Python environment, install `requirements.txt`, and launch JupyterLab from the project root.

**Run the notebooks with the project root as the working directory**, so relative paths such as `Raw_data`, `Processed_data`, and `coordinates.csv` resolve correctly. Running from *inside* `Processed_data/` may otherwise create or search for a nested `Processed_data/Processed_data/` path.

## Running the analysis

1. Place the authorized original `.set` recordings in a local `Raw_data/` directory. Keep their original subject filenames and any companion `.fdt` files.
2. Place `coordinates.csv` in the project root, with columns `EEG Label`, `X`, `Y`, and `Z`. Verify its reuse conditions before publishing it.
3. Run `Data_cleaning.ipynb` to generate cleaned `.fif` recordings in `Processed_data/`.
4. Run the four interpolation notebooks. They export subject-level evaluation metrics to CSV files in `Processed_data/`.
5. Ensure the interpolation notebooks also contain the additional `np.savez_compressed(...)` code for saving the original and reconstructed F3 signals to `Processed_data/Reconstructed_signals/`. This is required for signal-based figures in `Analysis.ipynb`.
6. Run `Analysis.ipynb`. It reads the previously saved CSV and `.npz` files and **does not recompute interpolation**.

The analysis notebook can still compare scalar metrics if only the CSV files are available. PSD, coherence, histograms, and log–log spectral figures additionally require saved signal `.npz` files.

## Evaluation and visualizations

The reconstruction notebooks calculate:

- root mean squared error (RMSE);
- Pearson correlation coefficient (and its p-value);
- mutual information;
- mean spectral coherence;
- Jensen–Shannon distance and divergence.

The separate analysis notebook visualizes method comparisons, power spectral density (PSD), frequency-dependent coherence, EEG amplitude histograms, and log–log PSD slopes. The histograms may show fitted normal-density curves as **visual references only**: this repository does not claim normality on that basis and does not perform formal normality testing.

## Reproducibility notes

- Subject identifiers, parameter settings, and output filenames follow the original thesis code.
- Data preprocessing and the four interpolation methods must run before the complete signal-based analysis can be reproduced.
- Results depend on the availability and preprocessing of the original EEG recordings and the electrode coordinates.
- The generated `.fif`, `.npz`, and most output files are not stored in Git to avoid committing large or potentially restricted data.
- The `CloughTocher2DInterpolator` method is a **two-dimensional piecewise cubic interpolator**, not a one-dimensional cubic spline.

## Research context

This code was developed for a bachelor's thesis on **EEG missing channel reconstruction using interpolation methods and comparative analysis**.

*Before public release, complete the dataset and coordinate-file provenance in the “Data and preprocessing” section. Add a thesis citation or permanent link if one is available.*
