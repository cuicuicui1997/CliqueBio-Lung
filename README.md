# CliqueBio-Lung minimal reproducibility package

This archive contains the smallest self-contained code subset needed to prepare the public inputs, run the CliqueBio-Lung pipeline, reproduce the experiments reported in the manuscript, verify the formal-consistency fixes, and regenerate the manuscript-facing tables and figures.

The archive intentionally excludes downloaded TCGA and STRING files, score caches, historical results, manuscript drafts, revision reports, and local development artifacts. Those items are either third-party data, derived outputs, or unnecessary for reproduction.

## 1. What this package reproduces

The package supports:

1. preparation of the pinned TCGA-LUAD expression, sample-label, and survival inputs used in the study;
2. preparation of the STRING v12.0 human physical protein-interaction network;
3. input validation and discovery-score caching;
4. the default CliqueBio-Lung LUAD run;
5. Experiments 1-10 from the manuscript workflow;
6. controlled baseline comparisons;
7. the small-pool reference-objective audit;
8. repeated subsampling and leakage-reduced discovery;
9. fixed-size clique-enumeration correctness tests;
10. five-`PYTHONHASHSEED` determinism regression; and
11. regeneration of manuscript-facing result tables and figures.

Exactness in this project refers to enumeration of all fixed-size clique seeds in the fixed reduced candidate graph and to the final induced-module clique-presence test. Candidate-graph construction, seed retention, module growth, and final redundancy-thresholded selection are not claimed to solve a global exact optimization problem.

## 2. Archive layout

```text
CliqueBio-Lung-minimal-1.0.0/
├── README.md
├── LICENSE
├── CITATION.cff
├── requirements.txt
├── environment.yml
├── MANIFEST.sha256
├── configs/
│   └── phase1_luad.yaml
├── data/
│   ├── download/                  # downloaded archives; initially empty
│   ├── processed/                 # generated caches and reports
│   └── raw/
│       └── lung_mechanism_gene_bank.tsv
├── scripts/
│   ├── prepare_tcga_luad_xena.py
│   ├── prepare_string_physical_ppi.py
│   ├── check_inputs.py
│   ├── build_score_cache.py
│   ├── run_phase1.py
│   ├── run_all_experiments.py
│   ├── exp01_...py through exp10_...py
│   └── supporting audit and plotting scripts
├── src/                           # core CliqueBio implementation
└── tests/
    └── test_formal_consistency.py
```

`MANIFEST.sha256` is generated when the release archive is built. It allows readers to verify that extracted files have not changed.

## 3. System requirements

The frozen development environment used:

- Python 3.14.3;
- Windows 11;
- the exact package versions in `requirements.txt`; and
- `PYTHONHASHSEED=2026` for the main replay.

The code is pure Python and should also run on recent Linux and macOS systems. Python 3.11-3.14 is recommended. Runtime values are machine-dependent; graph sizes, clique counts, selected module contents, structural metrics, and normalized scientific outputs are the reproducibility targets.

Recommended resources for the default run are:

- at least 8 GB RAM;
- at least 2 GB free disk space for downloads, processed inputs, caches, and outputs; and
- a multi-core CPU, although the reference scripts are primarily single-process.

The complete workflow can take several minutes or longer depending on hardware. The `r=5` scalability settings, repeated subsampling, and five-seed determinism regression are the slowest parts.

## 4. Installation

Run all commands from the extracted archive root.

### Option A: `venv` and `pip`

```bash
python -m venv .venv
```

Activate the environment:

```bash
# Linux or macOS
source .venv/bin/activate

# Windows PowerShell
.venv\Scripts\Activate.ps1
```

Install the frozen dependencies:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### Option B: Conda

```bash
conda env create -f environment.yml
conda activate cliquebio-lung
```

## 5. Data sources and downloads

No third-party TCGA or STRING data are redistributed in this archive. The preparation scripts download the exact public files used by the frozen workflow.

### 5.1 TCGA-LUAD expression and clinical data

The exact reproduction route uses the public UCSC Xena TCGA hub:

- TCGA hub landing page: <https://xenabrowser.net/datapages/?host=https%3A%2F%2Ftcga.xenahubs.net>
- expression matrix used by the script: <https://tcga.xenahubs.net/download/TCGA.LUAD.sampleMap/HiSeqV2.gz>
- clinical matrix used by the script: <https://tcga.xenahubs.net/download/TCGA.LUAD.sampleMap/LUAD_clinicalMatrix>

These files are approximately 30.9 MB and 0.7 MB, respectively. The Xena hub documents that this TCGA collection is based on the TCGA Data Coordinating Center and Broad Firehose releases available in 2016. The current NCI Genomic Data Commons project page is <https://portal.gdc.cancer.gov/projects/TCGA-LUAD>, but current GDC harmonized RNA-seq files are not a byte-for-byte substitute for the pinned Xena `HiSeqV2` matrix. Use the Xena route for exact count-level reproduction of this study.

The preparation script creates:

```text
data/raw/TCGA_LUAD_expression.tsv
data/raw/TCGA_LUAD_metadata.tsv
data/raw/TCGA_LUAD_survival.tsv
```

Users of TCGA data must follow applicable NIH/NCI data-use and attribution policies. Do not attempt to re-identify research participants. See <https://gdc.cancer.gov/access-data/data-access-policies>.

### 5.2 STRING physical protein interactions

The exact reproduction route uses the archived STRING v12.0 human files:

- STRING v12.0 download page: <https://version-12-0.string-db.org/cgi/download>
- human physical links: <https://stringdb-downloads.org/download/protein.physical.links.v12.0/9606.protein.physical.links.v12.0.txt.gz>
- human protein information: <https://stringdb-downloads.org/download/protein.info.v12.0/9606.protein.info.v12.0.txt.gz>
- STRING access and licensing: <https://version-12-0.string-db.org/cgi/access>

The two species-specific downloads are approximately 9.0 MB and 2.0 MB. STRING states that its downloadable data are available under CC BY 4.0. Cite STRING appropriately and clearly identify the symbol mapping and score filtering performed by this package.

The preparation script maps STRING protein identifiers to preferred gene symbols, removes self-loops, keeps the highest score for duplicate undirected symbol pairs, and applies the frozen `combined_score >= 700` threshold. It creates:

```text
data/raw/STRING_Homo_sapiens_physical_symbol.tsv
```

### 5.3 Curated mechanism-gene table

`data/raw/lung_mechanism_gene_bank.tsv` is included because it is a small, project-specific input required by the default knowledge-guided configuration. It contains the exact gene-category rows used by the frozen analysis. It is not presented as a comprehensive external database. Readers can regenerate the included file with:

```bash
python scripts/build_lung_mechanism_gene_bank.py
```

## 6. One-command reproduction

After installing dependencies, the following command downloads the pinned public inputs, validates them, builds the score cache, runs the pipeline and paper experiments, and regenerates result assets:

```bash
python scripts/run_all_experiments.py --prepare-data
```

To include the slower five-hash determinism regression:

```bash
python scripts/run_all_experiments.py --prepare-data --include-determinism
```

If data have already been prepared, omit `--prepare-data`:

```bash
python scripts/run_all_experiments.py
```

If a complete cache from Step 3 already exists, it can be reused to shorten repeated replays:

```bash
python scripts/run_all_experiments.py --reuse-cache
```

The runner validates the required cache files before continuing. Do not use this option with a cache built from different inputs or configuration settings.

To run experiments without regenerating manuscript-facing assets:

```bash
python scripts/run_all_experiments.py --skip-assets
```

The runner sets `PYTHONHASHSEED=2026` for every child process and passes the newly created default run directory to Experiment 10, avoiding any dependency on a historical timestamped path.

## 7. Step-by-step reproduction

The step-by-step route is useful for debugging and for recording intermediate checks.

### Step 1: download and prepare the public inputs

```bash
python scripts/prepare_tcga_luad_xena.py
python scripts/prepare_string_physical_ppi.py
```

The expected preparation summary is approximately:

| Item | Frozen value |
|---|---:|
| Expression genes | 20,530 |
| Expression samples | 576 |
| Tumor samples | 517 |
| Normal samples | 59 |
| Samples with usable survival fields | 563 |
| Processed STRING symbol-level edges | 86,519 |

If an upstream host changes a file, record its download date and checksum before interpreting differences.

### Step 2: validate inputs

```bash
python scripts/check_inputs.py --config configs/phase1_luad.yaml
```

In real-data mode, any critical input failure stops the workflow. The detailed report is written below `results/runs/*_input_check/logs/`.

### Step 3: build the discovery-score cache

```bash
python scripts/build_score_cache.py \
  --config configs/phase1_luad.yaml \
  --cache-dir data/processed/cache/phase1_luad_real
```

The cache stores node and edge score tables used by repeated graph-mining experiments. It is derived data and can always be rebuilt from the prepared inputs.

### Step 4: run the default pipeline

Linux or macOS:

```bash
PYTHONHASHSEED=2026 python scripts/run_phase1.py \
  --config configs/phase1_luad.yaml \
  --use-cache \
  --cache-dir data/processed/cache/phase1_luad_real
```

Windows PowerShell:

```powershell
$env:PYTHONHASHSEED = "2026"
python scripts/run_phase1.py `
  --config configs/phase1_luad.yaml `
  --use-cache `
  --cache-dir data/processed/cache/phase1_luad_real
```

The command prints the timestamped run directory. Keep that path for Experiment 10.

### Step 5: run manuscript Experiments 1-9

```bash
python scripts/exp01_problem_instance.py
python scripts/exp02_correctness.py
python scripts/exp03_runtime_baseline.py
python scripts/exp04_graph_size_scalability.py
python scripts/exp05_clique_size_scalability.py
python scripts/exp06_ppi_density_sensitivity.py
python scripts/exp07_candidate_reduction_ablation.py
python scripts/exp08_core_component_ablation.py
python scripts/exp09_module_construction_ablation.py
```

Each script accepts `--config` and `--cache-dir`. By default, outputs are written under `results/experiments/`.

### Step 6: run Experiment 10

Replace the placeholder with the directory printed by Step 4:

```bash
python scripts/exp10_biomedical_case_study.py \
  --run-dir results/runs/REPLACE_WITH_RUN_DIRECTORY
```

### Step 7: run the baseline and supplementary audits

```bash
python scripts/run_baseline_comparison.py
python scripts/run_objective_gap_small_instances.py
python scripts/run_robustness_bootstrap.py --runs 10 --fraction 0.8
python scripts/run_leakage_reduced.py
```

These commands write to `results/baselines/`, `results/objective_gap/`, `results/robustness/`, and `results/leakage_reduced/`.

### Step 8: regenerate result assets

Set the default run path when running this step manually:

```bash
export CLIQUEBIO_DEFAULT_RUN_DIR=results/runs/REPLACE_WITH_RUN_DIRECTORY
python scripts/generate_manuscript_assets.py
python scripts/make_experiment_tables.py
python scripts/figure_replot/redesign_fgcs_figures.py
```

In Windows PowerShell, use:

```powershell
$env:CLIQUEBIO_DEFAULT_RUN_DIR = "results/runs/REPLACE_WITH_RUN_DIRECTORY"
```

Generated tables and source data are placed under `manuscript_assets/`. Polished figure files are placed under `manuscript/src/figures/polished/`.

## 8. Experiment-to-script map

| Manuscript component | Entry point | Main output |
|---|---|---|
| Default CliqueBio-Lung run | `scripts/run_phase1.py` | `results/runs/*/` |
| Exp. 1: graph instances | `scripts/exp01_problem_instance.py` | `problem_instance_stats.csv` |
| Exp. 2: exact-set check | `scripts/exp02_correctness.py` | `correctness_comparison.csv` |
| Exp. 3: runtime comparison | `scripts/exp03_runtime_baseline.py` | `runtime_baseline.csv` |
| Exp. 4: graph-size scaling | `scripts/exp04_graph_size_scalability.py` | `graph_size_scalability.csv` |
| Exp. 5: clique-size scaling | `scripts/exp05_clique_size_scalability.py` | `clique_size_scalability.csv` |
| Exp. 6: STRING threshold | `scripts/exp06_ppi_density_sensitivity.py` | `ppi_density_sensitivity.csv` |
| Exp. 7: candidate reduction | `scripts/exp07_candidate_reduction_ablation.py` | `candidate_reduction_ablation.csv` |
| Exp. 8: core components | `scripts/exp08_core_component_ablation.py` | `core_component_ablation.csv` |
| Exp. 9: module construction | `scripts/exp09_module_construction_ablation.py` | `module_construction_ablation.csv` |
| Exp. 10: LUAD case study | `scripts/exp10_biomedical_case_study.py` | `biomedical_case_study_summary.csv` |
| Common-graph baselines | `scripts/run_baseline_comparison.py` | `baseline_summary.csv` |
| Reference-objective audit | `scripts/run_objective_gap_small_instances.py` | `objective_gap_small_instances.csv` |
| Subsampling stability | `scripts/run_robustness_bootstrap.py` | robustness CSV/JSON files |
| Leakage-reduced analysis | `scripts/run_leakage_reduced.py` | leakage-reduced CSV/JSON files |

## 9. Expected structural checkpoints

With the pinned inputs and default configuration, the principal non-runtime checkpoints are:

| Checkpoint | Expected value |
|---|---:|
| Default candidate-graph nodes | 949 |
| Default candidate-graph edges | 1,907 |
| Default degeneracy | 13 |
| Enumerated `r=3` clique seeds | 2,610 |
| Selected modules | 10 |
| Default mean structural clique coverage | 0.980 |
| Default mean pairwise Jaccard | approximately 0.033 |
| Five-seed determinism result | normalized artifacts byte-identical |

Wall-clock runtimes may differ substantially across machines. Small floating-point differences can arise across BLAS implementations, but they should not alter the documented canonical ordering or selected vertex sets.

## 10. Tests and determinism

Run the formal-consistency test suite:

```bash
python -m pytest -q
```

The included suite checks canonical ordering, exact clique presence, structural coverage, post-repair feasibility, and brute-force oracles for `r=3`, `r=4`, and `r=5`.

Run the lightweight enumeration self-check:

```bash
python scripts/self_check_clique_enum.py
```

Run the five-seed regression after building the cache:

```bash
python scripts/run_hash_seed_regression.py \
  --config configs/phase1_luad.yaml \
  --cache-dir data/processed/cache/phase1_luad_real \
  --output-dir results/determinism
```

This launches fresh subprocesses with `PYTHONHASHSEED=0,1,7,42,2026` and compares normalized scientific artifacts rather than timestamps or runtime fields.

## 11. Output interpretation

The default run creates:

```text
results/runs/<run_id>/
├── config_used.yaml
├── figures/
├── logs/
├── metrics/
├── modules/
└── seeds/
```

Important files include:

- `metrics/graph_stats.csv`: candidate-graph size and structure;
- `metrics/algorithm_trace.json`: enumeration and construction counters;
- `seeds/top_clique_seeds.csv`: retained fixed-size clique seeds;
- `modules/top_modules.csv`: selected provenance-aware candidate records and reported vertex modules;
- `metrics/module_evaluation.csv`: auxiliary within-cohort evaluation; and
- `config_used.yaml`: the complete configuration copied into the run.

Same-cohort AUC and survival summaries are auxiliary context, not independent validation. The returned modules are hypothesis-generating candidate modules rather than validated biomarkers or clinical predictors.

## 12. Troubleshooting

### A download URL fails

First retry the exact URL in Section 5. If an archived host is unavailable, download the named file from the corresponding official landing page and preserve its filename. Record the download date and SHA-256 checksum. Do not silently substitute current GDC RNA-seq processing for the pinned Xena `HiSeqV2` matrix.

### Input checking reports no sample overlap

Confirm that expression columns and metadata sample identifiers use the same 16-character TCGA sample barcode. The preparation script applies this rule automatically.

### The STRING edge count differs

Confirm all of the following:

- STRING version 12.0;
- NCBI taxonomy identifier 9606;
- `protein.physical.links`, not the general association network;
- preferred-name mapping from `protein.info.v12.0`;
- `combined_score >= 700`; and
- undirected duplicate removal after gene-symbol mapping.

### Runtime differs from the manuscript

Runtime depends on CPU, operating system, Python build, BLAS implementation, filesystem, and background load. Compare structural counts and normalized outputs before treating runtime differences as errors.

### A plotting script cannot find inputs

Run all experiment, baseline, robustness, and leakage-reduced commands first. For manual asset generation, set `CLIQUEBIO_DEFAULT_RUN_DIR` to the new default run directory.

### Windows path or multiprocessing issues

Use a short extraction path without non-ASCII characters, activate the environment in PowerShell, and run commands from the archive root.

## 13. Data and code availability language

Suggested manuscript wording:

> The TCGA-LUAD expression and clinical matrices used in this study are publicly available through the UCSC Xena TCGA hub, with the exact source URLs and processing script provided in the code archive. The current TCGA-LUAD project is also accessible through the NCI Genomic Data Commons. Human physical protein-interaction data were obtained from STRING v12.0; the exact species-specific files, filtering steps, and identifier mapping are documented in the archive. The project-specific mechanism-gene table, source code, configuration, and reproduction scripts are included with the software release. No controlled-access raw sequencing data are redistributed.

The public source repository is <https://github.com/cuicuicui1997/CliqueBio-Lung>. For a persistent scholarly citation, archive a numbered release in a DOI-granting repository such as Zenodo.

## 14. Citation and acknowledgement

If you use this package, cite the associated CliqueBio-Lung manuscript and the original TCGA, UCSC Xena, and STRING resources. The machine-readable `CITATION.cff` file can be updated with the article DOI after publication.

## 15. License

The software in this archive is released under the MIT License. Third-party datasets are governed by their own terms. In particular, STRING v12.0 data are distributed under CC BY 4.0, and TCGA users remain responsible for NIH/NCI data-use and attribution requirements.
