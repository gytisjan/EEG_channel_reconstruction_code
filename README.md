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

## Dataset and preprocessing

The EEG recordings used in this project come from **OpenNeuro dataset [ds004504](https://openneuro.org/datasets/ds004504)**, *A dataset of EEG recordings from: Alzheimer's disease, Frontotemporal dementia and Healthy subjects*. The data were collected at the **2nd Department of Neurology, AHEPA University Hospital, Thessaloniki, Greece**, and published by Miltiadous and colleagues.

The original dataset contains 88 resting-state, eyes-closed EEG recordings: 36 participants with Alzheimer's disease, 23 with frontotemporal dementia, and 29 cognitively normal healthy controls. This thesis uses **only the 29 healthy controls (`sub-037` to `sub-065`)**. The recordings were acquired with a Nihon Kohden EEG-2100 system using **19 scalp electrodes** positioned according to the international 10–20 system, sampled at **500 Hz**. The healthy-control recordings lasted approximately **13.8 minutes on average**.

The source dataset provides EEGLAB `.set` recordings and also includes separately preprocessed derivatives. This project implements its **own preprocessing** workflow. The `Data_cleaning.ipynb` notebook:

- removes boundary annotations without removing EEG samples;
- applies a **1–45 Hz** band-pass filter;
- sets an average EEG reference;
- fits a FastICA model and saves it along with the cleaned recording.

**ICA components are not removed by default** (`auto_remove_ica_components = False`). Thus, fitting ICA does not itself mean that artifact components were rejected from the exported cleaned signal.

The interpolation notebooks use electrode positions from `coordinates.csv` with columns `EEG Label`, `X`, `Y`, and `Z`. In the coordinate file used for this thesis, the **Fz position was not provided and was calculated separately**; the resulting fixed coordinates are defined in the interpolation notebooks. The exact external provenance of `coordinates.csv` and the procedure used to calculate Fz coordinates have **not yet been independently verified**. This coordinate file should be published only after confirming its source and reuse permissions.

The OpenNeuro dataset is distributed under the **CC0** license. The original EEG recordings are not included in this code repository; obtain them from the dataset page to reproduce the experiments. The dataset's CC0 license does not automatically establish the license of a separately sourced `coordinates.csv` file or of this project's code.

### Dataset references

- **Dataset:** Miltiadous, A., Tzimourta, K. D., Afrantou, T., Ioannidis, P., Grigoriadis, N., Tsalikakis, D. G., Angelidis, P., Tsipouras, M. G., Glavas, E., Giannakeas, N., & Tzallas, A. T. *A dataset of EEG recordings from: Alzheimer's disease, Frontotemporal dementia and Healthy subjects*. OpenNeuro, accession **ds004504**. https://openneuro.org/datasets/ds004504
- **Data descriptor:** Miltiadous, A., et al. (2023). *A Dataset of Scalp EEG Recordings of Alzheimer's Disease, Frontotemporal Dementia and Healthy Subjects from Routine EEG*. **Data, 8**(6), 95. https://doi.org/10.3390/data8060095

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

*The original EEG dataset is identified and cited above. The provenance of the separate coordinate file and the derivation of the Fz coordinates should still be verified. Add a thesis citation or permanent link if one is available.*
