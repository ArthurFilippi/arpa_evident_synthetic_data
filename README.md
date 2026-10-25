# ARPA-EVIDENT Synthetic Data

## Overview

This repository contains fully synthetic, multimodal data designed to simulate the ARPA-EVIDENT study workflow. It includes five fictional participants (`EV-001` through `EV-005`), with modality-specific acquisitions collected at different study time points.

No real participant data are included. The dataset is intended for development, demonstration, ingestion testing, and validation of the ARPA-EVIDENT data workflow.

Each acquisition is stored as a CSV file using the naming convention:

```text
{participant_id}_{modality}_{timepoint}.csv
```

The repository contains 80 primary acquisition files.

## Modalities

The examples below are a reproducible random sample of five feature names from each modality.

| Modality | Time points | Number of features | Five example features |
|---|---|---:|---|
| CyTOF | `D-2`, `D+2` | 1,000 | `CD8Tnaive_pMAPKAPK2_PMAIono`, `Th17_pP38_LPS`, `intMono_IkB_PMAIono`, `CD4Temra_pMAPKAPK2_Unstim`, `CD4Temra_pSTAT1_Unstim` |
| MRI | `D-2`, `D0`, `D+2` | 2,010 | `sgACC_Brainstem_NDI`, `Caudate_MT_corr`, `SMA_FbGainHit_beta`, `ACC_Pallidum_corr`, `Thalamus_Lingual_corr` |
| EEG | `D-2`, `D0`, `D+2` | 152 | `REM_theta_Cz_power`, `N2_sigma_O2_power`, `N1_beta_O2_power`, `REM_beta_F3_power`, `N3_alpha_O1_power` |
| PSG | `D-2`, `D0`, `D+2` | 248 | `N3_sigma_O1_power`, `N3_alpha_F4_power`, `adv_spindle_freq_F3`, `N2_theta_C4_power`, `N2_delta_C4_power` |
| Actigraphy | Daily from `D-2` through `D+2` | 100 | `phys_hr_rest`, `phone_longest_session_day`, `phys_counts_total`, `phys_resp_rate_night`, `geo_stationary_frac_24h` |

EEG files also contain `participant_id` and `timepoint` metadata columns; these are not counted among the 152 EEG features. MRI features represent DWI, resting-state, and MID-derived measurements.

## Intentionally injected errors

Four primary acquisition files contain deliberate errors that simulate common real-world data quality problems. Their machine-readable specification is available in [`INJECTED_ERRORS.csv`](INJECTED_ERRORS.csv).

| File | Injected error | Expected handling |
|---|---|---|
| `EV-002_psg_D0.csv` | Two numeric cells contain invalid text, including a decimal-comma value. | Reject as `non_numeric` and quarantine. |
| `EV-003_acti_D+1.csv` | The participant ID inside the file is `EV-005`, while its filename identifies `EV-003`. | Reject as `identity_mismatch` and quarantine. |
| `EV-004_eeg_D0.csv` | All 36 REM feature columns are absent. | Accept as a partial acquisition, retain `received` status, and flag `features_absent`. |
| `EV-005_cytof_D-2.csv` | One feature column name contains a typo. | Reject as `unknown_features` and quarantine. |

The `_edge_cases/` directory provides an additional ingestion-validation suite covering unknown columns, partial feature sets, non-numeric values, incorrect row counts, identity mismatches, equivalent duplicate files, changed-value conflicts and versioning, unplanned time points, wrong delimiters, incorrect modalities, and unknown participants. Expected outcomes are defined in [`_edge_cases/MANIFEST.csv`](_edge_cases/MANIFEST.csv).

## Intended use

The goal of this dataset is to test and demonstrate ingestion, completeness tracking, validation, versioning, quarantine, and audit behavior in the [EVIDENT Data Hub](https://arpa-dashboard-mvp.vercel.app/#).

Access to the Data Hub requires an authorized identifier and password. Credentials are not stored in this repository and should not be committed to source control.

