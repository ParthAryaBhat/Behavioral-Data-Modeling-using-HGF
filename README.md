# HGF-EEG Pipeline: Mechanistic Understanding of Predictive Coding

**BSE662 End-Semester Project | Team SynapticSpike | IIT Kanpur**

## Overview

This repository contains the complete analysis pipeline for fitting a 3-level Hierarchical Gaussian Filter (HGF) to a Roving MMN EEG paradigm and correlating the extracted prediction error signals with single-trial EEG amplitude across 33 participants.

---

## Pipeline Steps

```
Raw .set files  →  HGF (pyhgf)  →  EEG Extraction  →  LM/LMER  →  Plots
```

### Step 0: Prepare HGF Output
```bash
python3 python/01_prep_hgf_output.py
```
Renames columns from pyhgf output to match downstream scripts.

### Step 1: Extract EEG per Subject
```bash
# Via SLURM array (recommended)
sbatch slurm/01_extract_eeg_array.sh

# Or directly
python3 python/02_extract_all_subjects.py
```
Extracts Fz/Cz/Pz amplitude from each subject's `.set` file and merges with HGF output.

### Step 2: Concatenate Subjects
```bash
sbatch slurm/02_concat_subjects.sh
```
Combines all per-subject CSVs into `merged_eeg_hgf_ALL.csv`.

### Step 3: LM vs LMER Timewise Analysis
```bash
# Via SLURM (runs Fz, Cz, Pz in parallel)
export WORK_DIR=$HOME/rmmn
export N_PERM=500
sbatch slurm/03_submit_analysis.sh

# Or directly for one channel
EEG_CHANNEL=Fz N_PERM=500 Rscript R/03_lm_vs_lmer_analysis.R
```
Runs LM and LMER at every timepoint with cluster permutation correction (n=500).

### Step 4: O-LIFE Correlation
```bash
sbatch slurm/04_olife.sh
# Or: Rscript R/04_olife_correlation.R
```
Correlates MMN amplitude and eps2-EEG coupling with O-LIFE schizotypy scores.

---

## Repository Structure

```
├── python/
│   ├── 01_prep_hgf_output.py        # Prepare HGF CSV for analysis
│   └── 02_extract_all_subjects.py   # Extract EEG from .set files
├── R/
│   ├── 03_lm_vs_lmer_analysis.R     # Timewise LM vs LMER + cluster permutation
│   └── 04_olife_correlation.R       # O-LIFE schizotypy correlations
├── slurm/
│   ├── 01_extract_eeg_array.sh      # SLURM array: EEG extraction
│   ├── 02_concat_subjects.sh        # SLURM: concatenate subjects
│   ├── 03_submit_analysis.sh        # SLURM array: LM/LMER (Fz/Cz/Pz)
│   └── 04_olife.sh                  # SLURM: O-LIFE correlation
└── README.md
```

---

## Data Requirements

Place these files in `~/rmmn/` on the HPC:

| File | Description |
|------|-------------|
| `raw_set/*_sequence_epochs.set` | Preprocessed EEG epochs per subject |
| `trialwise_predictions.csv` | pyhgf HGF output |
| `behavioral_data_trialwise.csv` | Trial sequence (1392 trials) |
| `Survey_scores.csv` | O-LIFE, LSHS, ASI scores per subject |

---

## Expected Outputs

```
rmmn/results/
├── Fz/plots/
│   ├── eps2_LMvsLMER.png   # Sensory PE timewise effect at Fz
│   └── eps3_LMvsLMER.png   # Volatility PE timewise effect at Fz
├── Cz/plots/               # Same for Cz
├── Pz/plots/               # Same for Pz
└── olife/
    ├── MMN_vs_OLIFE.png
    ├── eps2_coupling_vs_OLIFE.png
    ├── grand_average_ERP.png
    └── subject_level_data.csv
```

---

## HPC Notes (IITK Param Sanganak)

- QOS: `pool_kotesrj_ra`
- Partition: `gpu` (only partition available for this account)
- **Do not run computation on the login node** — always use `sbatch`
- R packages installed via conda: `conda install -c conda-forge r-lme4 r-future r-future.apply`

---

## Key Scientific Findings

- **ε₂ (Sensory PE)**: Strong significant effects in MMN window (80-200ms) at Fz/Cz, and P300 window at Pz
- **ε₃ (Volatility PE)**: Late frontal effects only (300-400ms at Fz), consistent with hierarchical predictive coding
- **LM vs LMER**: Near-identical t-value timecourses — fixed effects dominate variance structure
- **O-LIFE**: SS1 Unusual Experiences correlates with LSHS (r=0.56) and ASI (r=0.64)

---

## Dependencies

**Python**: `pandas`, `numpy`, `scipy`

**R**: `dplyr`, `readr`, `ggplot2`, `tidyr`, `lme4`

**Install R packages via conda:**
```bash
conda install -c conda-forge r-lme4 r-future r-future.apply -y
Rscript -e "install.packages(c('dplyr','readr','ggplot2','tidyr','patchwork'), repos='https://cloud.r-project.org')"
```

---

## Team

| Member | Roll No | Contribution |
|--------|---------|-------------|
| Adwaaiit Pande | 230085 | HGF implementation, EEG pipeline, HPC scripting |
| Ayushi Mishra | 230275 | EEG preprocessing, Results write-up |
| Dhruv Shetty | 230370 | HPC management, LM/LMER scripting |
| Kshitiz Tyagi | 230585 | LMER formulation, plotting |
| Mukund Singhal | 230670 | Statistical interpretation, Introduction |
| Parth Arya Bhat | 230737 | ε₃ analysis, Conclusion |
| Vrutika Rao | 231177 | O-LIFE data collection, personality analysis |
