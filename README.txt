Synthetic EVIDENT dataset (no real participant).
5 patients EV-001..EV-005, one CSV per acquisition, named {id_patient}_{modality}_{timepoint}.csv.
  cytof  1000 features   D-2, D+2
  mri    2010 features   D-2, D0, D+2   (DWI, RS, MID)
  eeg     152 features   D-2, D0, D+2
  psg     248 features   D-2, D0, D+2
  acti    100 features   D-2 .. D+2, every day
80 files, of which 4 carry one injected error each unless generated with --clean (see INJECTED_ERRORS.csv).
_edge_cases/ holds files that must be refused, skipped or versioned; MANIFEST.csv says what to expect.
Regenerate with: python scripts/make_synthetic.py --out "<this folder>"
