# Replicating the LBP + HD Seizure Detector on the SWEC-ETHZ Short-term iEEG Dataset
# http://ieeg-swez.ethz.ch/
A Python 3.11 / PyTorch 2 replication of the **Local Binary Patterns + Hyperdimensional Computing (LBP+HD)** one-shot seizure detector by Burrello et al. (IEEE BioCAS 2018), packaged as a single Jupyter notebook that runs end to end on a CPU-only Windows machine.

> Part of the MSc Computer Science thesis **"Localization of Epileptic Seizures"**, Louisiana State University Shreveport.
> Author: **Mostafij Rahman**

---

## Highlights

- **Table I fully reproduced:** 100% specificity and 100% sensitivity for all 10 patients; mean detection delay **18.7 s** (paper: 18.2 s).
- **64 of 65 unseen seizures detected**, matching the paper.
- **Table II:** sensitivity identical to the paper; specificity matches or is close for most patients (see [Results](#results)).
- Original 2018 code ported from Python 2.7 to Python 3.11 / PyTorch 2, with **no GPU required**.
- A vectorised HD encoder that is **bit-identical** to the original sample-by-sample loop, but much faster.
- A training-index bug in the released code is fixed (and can be switched back on for comparison).

---

## Contents

- [Background](#background)
- [Requirements](#requirements)
- [Setup](#setup)
- [Dataset](#dataset)
- [Running the notebook](#running-the-notebook)
- [Settings](#settings)
- [Repository layout](#repository-layout)
- [Results](#results)
- [Changes from the original code](#changes-from-the-original-code)
- [Troubleshooting](#troubleshooting)
- [References](#references)

---

## Background

| | |
|---|---|
| **Paper** | A. Burrello, K. Schindler, L. Benini, A. Rahimi, *One-shot Learning for iEEG Seizure Detection Using End-to-end Binary Operations: Local Binary Patterns with Hyperdimensional Computing*, IEEE BioCAS 2018 ([arXiv:1809.01926](https://arxiv.org/abs/1809.01926)) |
| **Dataset** | SWEC-ETHZ iEEG Short-term Dataset — <http://ieeg-swez.ethz.ch/> |
| **Original code** | Released by the authors alongside the dataset (Short-term Dataset and Algorithms) |

The notebook `SWEC_LBPHD_replication.ipynb` repeats in one file every step originally run on the lab PC (VS Code terminal + scripts, 1–6 October 2026):

| Step | What it does |
|---|---|
| 1 | Settings and machine check |
| 2 | Project folders, VS Code settings, `.gitignore`, `requirements.txt` |
| 3 | Install Python packages (PyTorch, SciPy, …) |
| 4 | Git repository (optional) |
| 5 | Download dataset + original ETH code (optional; skipped if present) |
| 6 | Verify the dataset (16 patients, 100 seizure files) |
| 7 | Install the Python 3 / PyTorch 2 port of LBP+HD + self-test |
| 8 | Train and test LBP+HD for every patient (**long: ~2 days on CPU**) |
| 9 | Rebuild `summary.csv` + metrics under the paper's protocol |
| 10 | Compare with the paper's Tables I–II and the lab run; plot |

The ported LBP+HD code is embedded in the notebook (Step 7), so the dataset is the only thing that has to be obtained separately.

---

## Requirements

- **OS:** Windows 10/11 (other systems work; change `ROOT` in Step 1)
- **Python:** 3.11 (3.10 / 3.12 should also work; **not 3.13+**, since `numpy<2` has no wheels for it)
- **Editor:** VS Code with the Microsoft *Python* and *Jupyter* extensions, or plain Jupyter
- **Git:** optional, for change history
- **Disk:** ~12 GB for the unpacked data (+ ~12 GB if the notebook downloads the zips), plus ~1 GB for results
- **GPU:** optional. Without an NVIDIA GPU everything runs on the CPU (lab PC: no GPU, full run ≈ 2 days)

---

## Setup

Run in PowerShell.

**1. Install the software** (skip anything already installed), then reopen PowerShell:

```powershell
winget install -e --id Python.Python.3.11
winget install -e --id Microsoft.VisualStudioCode
winget install -e --id Git.Git
```

**2. Clone the repository** (or create the folder and copy `SWEC_LBPHD_replication.ipynb` and `README.md` into it):

```powershell
git clone <this-repo-url> C:\Users\<you>\sEEG
cd C:\Users\<you>\sEEG
```

**3. Create the Python environment:**

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip ipykernel notebook
```

The prompt should now start with `(.venv)`.

**4. (Git only) Set your identity once:**

```powershell
git config --global user.name  "Your Name"
git config --global user.email "you@example.com"
```

---

## Dataset

The dataset is **not included** in this repository (it is excluded by `.gitignore`). Expected location — one folder per patient, one `.mat` file per seizure:

```
data/raw/swec_ethz_short/ID1/Sz1.mat ... ID16/Sz6.mat
```

Two ways to get it:

1. **Copy** the folders `ID1` … `ID16` (100 `.mat` files, ~12 GB) from an existing copy, or
2. **Download automatically:** set `DOWNLOAD_DATA = True` in Step 1. Step 5 downloads the 19 zip files from ETH Zurich and unpacks them. `ID4a/4b`, `ID13a/b` and `ID14a/b` are merged into `ID4`, `ID13` and `ID14` automatically.

Seizure files per folder (verified in Step 6):

| ID1 | ID2 | ID3 | ID4 | ID5 | ID6 | ID7 | ID8 | ID9 | ID10 | ID11 | ID12 | ID13 | ID14 | ID15 | ID16 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 13 | 4 | 2 | 14 | 10 | 4 | 2 | 2 | 9 | 5 | 2 | 10 | 7 | 7 | 3 | 6 |

> **Patient numbering:** the scripts use the **paper's** patient numbers (P1–P16), which differ from the dataset folder numbers. The mapping (`Patient_trasformation` in `swec_config.py`) is detected automatically from the seizure counts — e.g. paper P6 = ID7, P3 = ID4, P10 = ID1. Step 7 prints `mapping: transform`.

---

## Running the notebook

### VS Code

1. **File → Open Folder** → the project folder
2. Open `SWEC_LBPHD_replication.ipynb`
3. **Select Kernel** (top right) → *Python Environments* → `.venv` (Python 3.11)
4. Run cells top to bottom (`Shift+Enter`) or **Run All**

### Jupyter

```powershell
.\.venv\Scripts\Activate.ps1
jupyter notebook SWEC_LBPHD_replication.ipynb
```

### Unattended (no open editor)

```powershell
.\.venv\Scripts\Activate.ps1
jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=-1 SWEC_LBPHD_replication.ipynb
```

Restart the command once after Step 3 has installed PyTorch for the first time.

### Important notes

- After Step 3 installs PyTorch/NumPy for the first time, **restart the kernel** and rerun Step 1. After **any** kernel restart, run Step 1 first.
- **Step 8a** is a smoke test on patient 6 (~10 min on CPU). Expected: specificity 100, sensitivity 100, mean delay ≈ 6.25 s (paper: 6.3 s).
- **Step 8b** is the long run. Keep the PC awake (*Settings → System → Power → Sleep: Never*). It is resumable: patients whose predictions are complete are skipped.
- Don't open `summary.csv` in Excel while Step 8 or 9 is running — Excel locks the file.

---

## Settings

Set in the first code cell of Step 1:

| Setting | Meaning | Default |
|---|---|---|
| `ROOT` | Project folder | `C:\Users\mrahman\sEEG` |
| `DOWNLOAD_DATA` | Download the dataset zips | `False` |
| `DOWNLOAD_CODE` | Download the original ETH code (reference only) | `True` |
| `FOLD_MODE` | `"full"` = exactly the lab run: sliding k-fold for every patient, 679 seizure tests, ~2 days on CPU<br>`"original"` = only what the paper's tables need, 246 tests, ~2.8× faster; identical paper-protocol results | `"full"` |
| `SEED` | Hypervector random seed (authors' value) | `1` |
| `RUN_PATIENTS` | Paper patient numbers to run (smallest first) | all 16 |
| `RERUN_EXISTING` | Re-run finished patients | `False` |
| `USE_GIT` | Keep a Git history | `True` |

The scripts also read these environment variables (set by the notebook):

| Variable | Purpose |
|---|---|
| `SWEC_ROOT` | Project root |
| `SWEC_FOLDS` | Fold mode |
| `SWEC_SEED` | Random seed |
| `SWEC_PATIENT` | `"6"`, `"1,2,3"` or `"all"` |
| `SWEC_GPU` | GPU index |
| `SWEC_ID_MAP` | Force `direct` \| `transform` \| `inverse` patient mapping |
| `SWEC_ORIGINAL_TRAIN_INDEX=1` | Reproduce the original training-index bug |
| `SWEC_FAST=0` | Use the original (slower) encoder; results identical |

---

## Repository layout

Folders marked † are created at run time and excluded from Git.

```
sEEG/
├── SWEC_LBPHD_replication.ipynb        # the notebook (Steps 1–10)
├── README.md                           # this file
├── requirements.txt                    # Step 2
├── .gitignore                          # Step 2; excludes data, .venv, results
├── .vscode/
│   ├── settings.json                   # Step 2
│   └── launch.json                     # Step 2; F5 runs a script for patient 6
├── .venv/ †                            # Python 3.11 environment
├── data/ †
│   └── raw/swec_ethz_short/
│       ├── ID1/ … ID16/                # Sz*.mat seizure files
│       └── _zips/                      # downloaded zips (Step 5 only)
├── code/
│   └── swec_ethz_short/
│       ├── LBP+HD/LBP+HD/              # scripts run from here (Steps 7–8)
│       │   ├── iEEGClassifierLBP+HD.py     # ported training + testing
│       │   ├── iEEGFunctions.py            # ported HD encoder (LBP + HD)
│       │   ├── HD_PostProcessing.py        # ported post-processing
│       │   ├── swec_config.py              # shared paths/tables/settings (new)
│       │   ├── paper_protocol_metrics.py   # paper-protocol metrics (new)
│       │   └── README.md                   # authors' original (if downloaded)
│       ├── baselines/                  # authors' LBP+SVM / LGP+MLP (not ported yet)
│       ├── LBP+HD_Identification/      # authors' TBME 2019 code (not ported yet)
│       └── _zips/                      # original code zips (Step 5)
├── scripts/
│   └── verify_swec_short.py            # dataset checker + loader (Step 6)
└── results/ †
    ├── swec_short_inventory.csv        # one row per seizure file (Step 6)
    ├── lbphd_P<n>.log                  # console output per patient (Step 8)
    ├── postprocessing_all.log          # Step 9
    ├── paper_protocol_metrics.log      # Step 9
    ├── plots/
    │   ├── ID1_Sz1.png                 # example recording (Step 6)
    │   └── lbphd_vs_paper.png          # replication vs paper (Step 10)
    └── swec_short/LBP+HD/
        ├── predictions/Pat1 … Pat16/HDClassifier_Seizure<j>Cross<k>.pt
        ├── summary.csv                             # released-code metrics (Step 9)
        └── LBPHD_replication_vs_BioCAS2018.csv     # paper vs replication (Step 9)
```

---

## Results

### Two result tables

| File | Definition | Use |
|---|---|---|
| `summary.csv` | Released-code metrics (what `HD_PostProcessing.py` prints): averages over every recording in every fold, **including** seizures used for training | Comparison with the released code |
| `LBPHD_replication_vs_BioCAS2018.csv` | **Paper protocol** — unseen seizures only. Table I patients (1, 2, 4, 5, 6, 8, 11, 13, 15, 16): k-fold. Table II patients (3, 7, 9, 10, 12, 14): single split — train on the first *m* seizures, test on the rest. Columns: `paper_*` = published, `repl_*` = this replication, `released_*` = released-code metrics | **Thesis / reporting** |

### Lab PC run (3 October 2026, `FOLD_MODE="full"`, `SEED=1`, CPU)

Step 10 compares your numbers with these automatically.

**Table I** — k-fold

| Patient | Paper delay (s) | Repl. delay (s) | Paper spec. (%) | Repl. spec. (%) | Paper sens. (%) | Repl. sens. (%) |
|---|---:|---:|---:|---:|---:|---:|
| P1 | 6.3 | 7.21 | 100 | 100 | 100 | 100 |
| P2 | 15.1 | 16.04 | 100 | 100 | 100 | 100 |
| P4 | 34.5 | 34.04 | 100 | 100 | 100 | 100 |
| P5 | 20.9 | 25.03 | 100 | 100 | 100 | 100 |
| P6 | 6.3 | 6.25 | 100 | 100 | 100 | 100 |
| P8 | 13.2 | 13.58 | 100 | 100 | 100 | 100 |
| P11 | 7.0 | 7.25 | 100 | 100 | 100 | 100 |
| P13 | 10.0 | 10.25 | 100 | 100 | 100 | 100 |
| P15 | 36.4 | 36.04 | 100 | 100 | 100 | 100 |
| P16 | 32.3 | 31.00 | 100 | 100 | 100 | 100 |
| **Mean** | **18.2** | **18.7** | **100** | **100** | **100** | **100** |

**Table II** — single split

| Patient | Paper delay (s) | Repl. delay (s) | Paper spec. (%) | Repl. spec. (%) | Paper sens. (%) | Repl. sens. (%) |
|---|---:|---:|---:|---:|---:|---:|
| P3 | 21.8 | 15.91 | 94.17 | 87.19 | 100 | 100 |
| P7 | 5.0 | −9.38 | 44.00 | 11.62 | 100 | 100 |
| P9 | 16.2 | 15.67 | 100.0 | 100.0 | 100 | 100 |
| P10 | 9.3 | 4.11 | 99.47 | 98.32 | 90 | 90 |
| P12 | 12.6 | 0.25 | 93.10 | 71.40 | 100 | 100 |
| P14 | 21.7 | 5.42 | 100.0 | 97.60 | 100 | 100 |

### Summary

- **Table I: fully reproduced.** 100% specificity and sensitivity for all 10 patients; mean delay 18.7 s vs 18.2 s in the paper.
- **Table II:** sensitivity identical to the paper. Specificity matches for P9 and is close for P10 and P14, but lower for P3, P12 and P7.
- **64 of 65 unseen seizures detected**, the same as the paper.
- **Likely cause of the Table II gap:** the random hypervectors differ from the authors' (PyTorch 2 vs the 2018 version, same seed), while the per-patient thresholds were tuned to the authors' vectors.

### Next step

Rerun the Table II patients with several seeds:

```python
FOLD_MODE    = "original"
RUN_PATIENTS = [3, 7, 9, 10, 12, 14]
SEED         = 2   # then 3, 4, ...
```

Copy `results/swec_short/LBP+HD` aside between seeds so runs don't overwrite each other.

---

## Changes from the original code

All changes are marked `PORT:` in the code.

1. **Python 2.7 → 3.11 and old PyTorch → PyTorch 2;** runs on CPU or any GPU (the original required CUDA GPU #1 of the ETH server).
2. **Unified configuration:** the released training and post-processing scripts used different file names, folders and fold counts and could not run together; they now share `swec_config.py`.
3. **Bug fix:** from the 2nd fold on, training read seizure `i+1+cross` but took the ictal times of seizure `i+1`. Fixed; `SWEC_ORIGINAL_TRAIN_INDEX=1` restores the original behaviour.
4. **Patient numbering resolved** (paper numbers vs dataset folders), checked against the seizure counts.
5. **Vectorised HD encoder** with bit-identical output to the original sample-by-sample loop (checked in Step 7); `SWEC_FAST=0` uses the original.
6. **Cleaner logging and robustness:** one progress line per seizure instead of one per half second; division by zero guarded when nothing is detected.
7. **Metrics:** `summary.csv` keeps the released definition; the paper protocol (unseen seizures only, single split for Table II) is computed separately by `paper_protocol_metrics.py`.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `No module named scipy` / packages install into Python 3.13 | The notebook kernel is not the `.venv`. Select the `.venv` kernel and rerun Step 3. |
| `Found no NVIDIA driver on your system` | Only happens with the **original** scripts (`torch.cuda.set_device`). The ported scripts fall back to CPU. Rerun Step 7 to restore them. |
| `PermissionError … summary.csv` | `summary.csv` is open in Excel. Close it and rerun Step 9 — predictions are saved, nothing is lost. |
| `git is not recognized` | Install Git and restart VS Code, or set `USE_GIT = False`. |
| A file named `iEEGClassifierLBP_HD.py` appears | A browser renamed `+` to `_` during download. Step 7 writes the correct name; delete the `_HD` copy. |
| `Seizure-file counts … do not match` | Data missing or merged incorrectly. Check Step 6 and the [dataset table](#dataset). |
| Kernel died / PC restarted during Step 8 | Run Step 1, then Step 8b again — finished patients are skipped. |
| NumPy 2.x found | `pip install "numpy<2"` in the `.venv`, then restart the kernel. |

---

## References

If you use this replication, please cite the original paper:

```bibtex
@inproceedings{burrello2018oneshot,
  author    = {Burrello, Alessio and Schindler, Kaspar and Benini, Luca and Rahimi, Abbas},
  title     = {One-shot Learning for {iEEG} Seizure Detection Using End-to-end Binary Operations:
               Local Binary Patterns with Hyperdimensional Computing},
  booktitle = {IEEE Biomedical Circuits and Systems Conference (BioCAS)},
  year      = {2018},
  note      = {arXiv:1809.01926}
}
```

- SWEC-ETHZ iEEG Database: <http://ieeg-swez.ethz.ch/>
- Paper preprint: <https://arxiv.org/abs/1809.01926>

The dataset and the original LBP+HD code belong to their authors; check the terms on the SWEC-ETHZ website before redistributing either.
