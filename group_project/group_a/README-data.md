# Group A — Yogurt Study: Data Summary

This document is a quick-reference guide to the yogurt study dataset in
`data/group_a_yogurt/`. Use it to remind yourself what's in each file and
how they connect.

---

## Study overview

> **Question:** Does eating yogurt change the vaginal microbiome?

- **51 participants** assigned to either a **yogurt** group or an **unchanged
  diet** group
- Samples taken at **baseline** (before the study started) and **after
  antibiotic** treatment
- Each participant was sampled at both time points, for a total of **102
  samples**

---

## File 1: `00_sample_ids_yogurt.csv`

**What one row represents:** one biological sample (one swab from one person
at one visit).

| Column | Meaning | Possible values |
|--------|---------|-----------------|
| `pid` | Participant ID | `pid_01` ... `pid_51` |
| `time_point` | When the sample was taken | `baseline`, `after_antibiotic` |
| `arm` | Study group | `yogurt`, `unchanged_diet` |
| `sample_id` | Unique sample ID | `YOG001` ... `UNCH092` |

**Size:** 102 data rows (51 participants × 2 time points).

---

## File 2: `01_participant_metadata_yogurt.csv`

**What one row represents:** one participant, with their background info.

| Column | Meaning | Possible values |
|--------|---------|-----------------|
| `pid` | Participant ID | `pid_01` ... `pid_51` |
| `arm` | Study group | `yogurt`, `unchanged_diet` |
| `days_since_last_sex` | Days since last sex | 0–7 |
| `birth_control` | Type of birth control | `Depoprovera`, `no hormonal birth control` |
| `age` | Age in years | 24–37 |
| `education` | Education level | `less than grade 9`, `grade 10-12, not matriculated`, `grade 10-12, matriculated`, `post-secondary` |
| `sex` | Biological sex | all `female` |

**Size:** 51 rows (one per participant).

**Note:** `days_since_last_sex` has some missing values — check for `NA`.

---

## File 3: `02_qpcr_results_yogurt.csv`

**What one row represents:** bacterial DNA counts from one sample. qPCR is a
lab test that counts how many copies of bacterial DNA are present.

| Column | Meaning |
|--------|---------|
| `sample_id` | Links to `00_sample_ids_yogurt.csv` |
| `qpcr_bacteria` | Total bacterial DNA copies (all bacteria combined) |
| `qpcr_crispatus` | DNA copies of *Lactobacillus crispatus* — a "good" bacterium linked to a healthy vagina |
| `qpcr_iners` | DNA copies of *Lactobacillus iners* — another *Lactobacillus* species, often linked to a less stable microbiome |

**Size:** 102 rows (one per sample).

**Watch out for:** Some values are `0` — that means the bacterium wasn't
detected at all.

---

## File 4: `03_luminex_results_yogurt.csv`

**What one row represents:** one cytokine measurement for one sample. Each
sample has 10 rows (one per cytokine). **Cytokines** are signalling proteins
the immune system uses — they tell us about inflammation.

| Column | Meaning | Possible values |
|--------|---------|-----------------|
| `sample_id` | Links to `00_sample_ids_yogurt.csv` | |
| `cytokine` | Which cytokine was measured | `IL-1a`, `IL-1b`, `IL-6`, `IL-8`, `IL-10`, `TNFa`, `IFN-Y`, `IP-10`, `MIG`, `MIP-3a` |
| `conc` | Concentration in pg/mL | |
| `limits` | Was the measurement reliable? | `within limits`, `out of range` |

**Size:** 1,020 rows (102 samples × 10 cytokines).

**Watch out for:** Measurements that are `out of range` — the concentration
was too low or too high to measure accurately. You might want to filter them
out before plotting.

---

## How the tables connect

```
01_participant_metadata ──pid──► 00_sample_ids ──sample_id──► 02_qpcr_results
                                      │
                                      └──sample_id──► 03_luminex_results
```

- `pid` joins metadata ↔ sample IDs
- `sample_id` joins sample IDs ↔ qPCR results and Luminex results

---

## Quick R code

```r
library(tidyverse)

sample_ids  <- read_csv("data/group_a_yogurt/00_sample_ids_yogurt.csv")
participants <- read_csv("data/group_a_yogurt/01_participant_metadata_yogurt.csv")
qpcr        <- read_csv("data/group_a_yogurt/02_qpcr_results_yogurt.csv")
luminex     <- read_csv("data/group_a_yogurt/03_luminex_results_yogurt.csv")

# Join everything together
yogurt_data <- sample_ids |>
  left_join(participants, by = c("pid", "arm")) |>
  left_join(qpcr, by = "sample_id") |>
  left_join(luminex, by = "sample_id")

# Quick check: how many participants in each group?
participants |> count(arm)
```

---

## Questions you might investigate

- Does the yogurt group have higher *L. crispatus* (the "good" bacterium)
  after antibiotics?
- Are cytokine levels (inflammation markers) different between the two
  groups?
- How does total bacterial load change from baseline to after antibiotic in
  each group?
- Do baseline characteristics (age, birth control) differ between groups?