# ARPA-EVIDENT Synthetic Data

## Overview

This repository contains fully synthetic, multimodal data designed to simulate the ARPA-EVIDENT study workflow on its real timepoints. It includes 143 fictional participants (`ARPA_001` through `ARPA_143`), with modality-specific acquisitions collected at the study days of the protocol. Each acquisition is stored as a CSV file named `{participant_id}_{modality}_{timepoint}.csv` (CyTOF has no timepoint in its name: one file holds the three timepoints of a patient).

The repository contains 10,516 primary acquisition files. A patient only has the files of the days it has lived (`last_study_day` in [`roster.csv`](roster.csv)): 87 patients have lived the whole study (D+28), the other 56 are spread over the study, so that every day has patients. About 0.4 % of the cells are empty (not measured), and half of the one-row files list their columns in another order, as different instruments do. Everything is a pure function of (patient, modality, study day): regenerating gives the same bytes.

## Modalities

The examples below are a reproducible sample of five feature names from each modality.

| Modality | Time points | Files | Number of features | Five example features | Naming |
|---|---|---:|---:|---|---|
| CyTOF | `D-7`, `D0`, `D+7` (t1, t2, t3) in one file, sent once D+7 is done | 114 | 1,000 | `NKbright_pP38_IFNa`, `CD8Tnaive_pSTAT3_Unstim`, `Th17_pNFkB_Unstim`, `Basophils_pNFkB_PMAIono`, `Basophils_pSTAT5_IFNa` | `ARPA_012_cytof.csv` |
| MRI | `D-7`, `D+7` | 248 | 2,010 | `Putamen_NAcc_corr`, `TPJ_AntLoss1_beta`, `SupramarginalGyrus_Operculum_corr`, `Habenula_AntLoss5_beta`, `Amygdala_Hypothalamus_corr` | `ARPA_012_mri_D-7.csv` |
| EEG | daily from `D-14` through `D+28` | 4,953 | 50 | `N2_sigma_O2_power`, `N3_beta_O2_power`, `N2_delta_Cz_power`, `N2_theta_Cz_power`, `pct_N1` | `ARPA_012_eeg_D-14.csv` |
| PSG | `D-7`, `D+7` | 248 | 248 | `REM_sigma_C4_power`, `REM_delta_F3_power`, `N2_delta_C4_power`, `N3_sigma_O2_power`, `ECG_REM_SDNN` | `ARPA_012_psg_D-7.csv` |
| Actigraphy | daily from `D-14` through `D+28` | 4,953 | 50 | `phys_SOL`, `phone_calls_n_night`, `phys_hr_max`, `geo_transitions_24h`, `geo_n_places_night` | `ARPA_012_acti_D-14.csv` |

CyTOF is a long table (population, reagent, stimulation, sampleID, group, time, feature, one row per feature and timepoint); the other modalities are one row of features per file.

## Intentionally injected errors

1057 files out of 10,516 (10.1%) contain one deliberate error, graded from a warning to an unreadable file: 617 minor, 343 moderate, 97 major. Their machine-readable specification (file, kind, severity, expected status and flags) is available in [`INJECTED_ERRORS.csv`](INJECTED_ERRORS.csv). The hub never discards a file: it stores it, flags it, and the status tells how far it can be trusted.

| Severity | Meaning | Expected handling |
|---|---|---|
| minor | The file is read; something small is wrong. | Accepted, flagged as a warning. Status `validated`, or `received` when features are absent (stored as received, not validated). |
| moderate | A readable file with an error flag. | Accepted, flagged as an error, status `qc_failed` ("failed check"): kept out of the analysis views until replaced. |
| major | The file cannot be read as a table of the contract, or it names another patient. | Accepted, flagged as an error, status `qc_failed`. |

### One-row files (MRI, PSG, EEG, actigraphy)

| Kind | Severity | Files | Injected error | Expected flags | Expected status |
|---|---|---:|---|---|---|
| `unknown_column` | minor | 262 | an extra column that is no feature (`operator_note`) | `unknown_features` | `validated` |
| `column_typo` | minor | 190 | one feature name carries a typo (an `x` at the end): unknown column, and the real feature is absent | `unknown_features`, `features_absent` | `received` |
| `partial_columns` | minor | 158 | one feature column in five is missing (20 % absent, under the 40 % limit) | `features_absent` | `received` |
| `non_numeric` | moderate | 160 | two cells hold text (`n.d.`) or a decimal comma (`1,5`) | `non_numeric` | `qc_failed` |
| `too_much_missing` | moderate | 113 | 55 % of the feature columns are missing (over the 40 % limit) | `too_much_missing`, `features_absent` | `qc_failed` |
| `duplicate_column` | moderate | 67 | the same feature appears in two columns | `duplicate_features` | `qc_failed` |
| `identity_column` | major | 35 | a `participant_id` column names another patient than the file name and the form | `identity_mismatch` | `qc_failed` |
| `wrong_delimiter` | major | 32 | separated by `;` instead of commas: the file cannot be read as a table | `delimiter` | `qc_failed` |
| `two_rows` | major | 21 | two data rows where an acquisition file has exactly one | `row_count` | `qc_failed` |
| `empty_file` | major | 7 | a file of zero bytes | `empty` | `qc_failed` |

### CyTOF files

| Kind | Severity | Files | Injected error | Expected flags | Expected status |
|---|---|---:|---|---|---|
| `timepoint_missing` | minor | 3 | the last timepoint (t3) is not in the file | `timepoint_absent` | `validated` |
| `timepoint_partial` | minor | 3 | 10 % of the features of one timepoint are missing | `features_absent` | `received` |
| `unknown_feature` | minor | 1 | one population / marker / stimulation that is not in the reference | `unknown_features` | `validated` |
| `too_much_missing_t` | moderate | 1 | 70 % of the features of one timepoint are missing | `too_much_missing`, `features_absent` | `qc_failed` |
| `non_numeric_t` | moderate | 1 | two values of one timepoint hold text | `non_numeric` | `qc_failed` |
| `column_typo_t` | moderate | 1 | the column `population` is written `populations`: the contract cannot be applied | `missing_columns` | `qc_failed` |
| `identity_sample` | major | 1 | `sampleID` names another patient than the file | `identity_mismatch` | `qc_failed` |
| `wrong_delimiter_t` | major | 1 | separated by `;` instead of commas | `delimiter` | `qc_failed` |

Flawed files per modality: CyTOF 12 of 114, MRI 29 of 248, PSG 25 of 248, EEG 508 of 4,953, Actigraphy 483 of 4,953.

The hub's tests (`tests/test_error_catalogue.py`) force every kind on one file and check the answer, so this table is verified, not only written.

The `_edge_cases/` directory provides an additional ingestion-validation suite covering unknown columns, partial feature sets, non-numeric values, incorrect row counts, identity mismatches, equivalent duplicate files, changed-value conflicts and versioning, unplanned time points, wrong delimiters, incorrect modalities, and unknown participants. Expected outcomes are defined in [`_edge_cases/MANIFEST.csv`](_edge_cases/MANIFEST.csv).

## Intended use

The goal of this dataset is to test and demonstrate ingestion, completeness tracking, validation, versioning, audit and export behavior in the [EVIDENT Data Hub](https://arpa-dashboard-mvp.vercel.app/#). Put this folder in `HUB_SIMULATION_DIR` (or run `hub simulate`) and the hub plays the files in study order, as the laboratories would send them. After the whole run, the flags of the ledger and the Overview must show exactly the lines of `INJECTED_ERRORS.csv`; anything else is a bug.

Access to the Data Hub requires an authorized identifier and password. Credentials are not stored in this repository and should not be committed to source control.

## Regenerate

```
python scripts/make_synthetic.py --out "<this folder>"    # files, INJECTED_ERRORS.csv, roster.csv, this README
python scripts/make_synthetic.py --out DIR --clean           # the same cohort without any error
```
