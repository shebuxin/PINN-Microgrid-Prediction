# Microgrid 3-Phase AI Security Prediction Course - Reviewed Release

This package contains the reviewed research design and the complete 8-week teaching materials for:

**Physics-informed / phase-aware AI for fast static N-1 security screening in unbalanced microgrids.**

## How to use this package

- Use `*_clean*.ipynb` notebooks for teaching and student hands-on work.
- Use `*_executed*.ipynb` notebooks as instructor reference copies with expected outputs.
- Use `slides/*.pdf` for lectures and `slides/*.tex` for Beamer editing.
- Use `outputs/` folders for generated datasets, validation summaries, plots, and paper-ready tables.
- Start with `00_overview/overall_review_report.md` and `00_overview/overall_review_validation_summary.csv`.

## Reproducible environment

Create the tested environment from the repository root, then register it as a
Jupyter kernel:

```bash
micromamba create -f environment.yml -y
micromamba run -n pdpower python -m ipykernel install --user \
  --name pdpower --display-name "Python (pdpower)"
```

The release audit uses Python 3.11, pandapower 3.2.1, NumPy 1.26.4,
pandas 2.2.3, SciPy 1.13.1, scikit-learn 1.8, and PyTorch 2.10. The Week 7
notebook implements message passing in plain PyTorch and does not require
PyTorch Geometric.

## Folder map

```text
00_overview/                         Review report, validation summary, file manifest
01_research_design/                  Research plan, paper plan, 8-week syllabus
01_week01_microgrid_pandapower_intro/
02_week02_three_phase_power_flow/
03_week03_microgrid_scenario_generation/
04_week04_three_phase_nminus1_labeling/
05_week05_ml_baselines_screening/
06_week06_physics_informed_mlp/
07_week07_phase_aware_gnn_screening/
08_week08_paper_reproducibility/
90_paper_package/                    Mini-paper skeleton, tables, figures
91_presentation_package/             Final presentation outline
98_original_week_zip_archives/        Original week-level material archives
```

## Validation status

The release audit executes the clean notebooks from a fresh repository copy,
checks clean/executed source parity, validates weekly artifacts and formulas,
and recompiles the LaTeX sources. See the overview report for the current
results instead of relying on old page counts or cached notebook outputs.

## Important interpretation note

The current dataset is a compact teaching benchmark: 7 scenarios x 11 contingencies = 77 N-1 samples on a small radial teaching feeder. This is appropriate for instruction, proof-of-concept, and mini-paper drafting, but it should not be presented as deployment-level evidence. A research paper should scale the study to more scenarios, more feeders, and stronger out-of-distribution testing.

## Combined slides

A merged PDF of all Week 1-8 Beamer slides is available at `92_combined_slides/all_weeks_slides_combined.pdf` for quick review.
