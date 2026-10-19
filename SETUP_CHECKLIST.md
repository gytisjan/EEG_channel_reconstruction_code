# GitHub repository setup checklist

This checklist is for local setup and does not need to be uploaded to GitHub.

- [ ] Confirm all six notebooks are in the repository root and use the final names in README.md.
- [ ] Add README.md and requirements.txt to the repository root.
- [ ] Add the file `.gitignore` (starting with a dot) to the repository root.
- [ ] Verify whether `coordinates.csv` may be redistributed; if yes, upload it with the exact required column names `EEG Label`, `X`, `Y`, and `Z`.
- [ ] Do not commit `.set`, `.fdt`, `.fif`, or `.npz` files.
- [ ] Keep local directories `Raw_data/` and `Processed_data/` next to the notebooks.
- [ ] Start JupyterLab from the project root, not from inside Processed_data/.
- [ ] Confirm the four interpolation notebooks save reconstructed signals to `Processed_data/Reconstructed_signals/` if all Analysis.ipynb figures are needed.
- [x] Identify and cite the original EEG dataset (OpenNeuro ds004504, CC0) in README.md.
- [ ] Verify the separate `coordinates.csv` source and document how the missing Fz coordinates were calculated.
- [ ] Check the file history for any previously committed datasets before switching visibility from Private to Public; .gitignore does not remove existing history.
- [ ] Decide whether to publish a small, permission-safe selection of final summary results.
- [ ] Optionally add an appropriate license for original code after deciding reuse terms.
- [ ] Make the repository Public only after the data and notebook outputs have been reviewed for restricted information.
