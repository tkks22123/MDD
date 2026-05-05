# PDRA 91336 Technical Task — Xiaoke

This repository contains the submission for the Post Doctoral Research
Associate (91336) technical task at the Liverpool School of Tropical
Medicine, MalariaGEN Vector Observatory.

## Contents

| File | Description |
|:-----|:------------|
| `admixture_mixin/anoph/admixture.py` | Mixin class providing admixture analysis via ezstructure |
| `tests/anoph/test_admixture.py` | Unit tests (mocked, no external toolchain required) |
| `part1a_justification.md` | Tool selection justification (Part 1a) |
| `part2_improvement.md` | Package-wide improvement proposal (Part 2) |
| `environment.yml` | Conda environment specification |
| `README.md` | This file |

## Quick Start

### 1. Create and activate the conda environment

```bash
conda env create -f environment.yml
conda activate pdra_test
```

### 2. Run the tests

**Windows (PowerShell):**

```powershell
$env:PYTHONPATH="."
pytest tests/anoph/test_admixture.py -v
```

> **Expected output:** 3 passed

## How the tests satisfy Part 1b requirements

| Task Requirement | Test Case | How it is verified |
|:-----------------|:----------|:-------------------|
| **Expected inputs** (minimum samples) | `test_guard_k_exceeds_sample_count` | Raises `ValueError` when K >= sample count |
| **Defensive programming** (empty inputs) | `test_guard_empty_samples` | Raises `ValueError` when no samples match query |
| **Reproducibility** | `test_full_pipeline_with_mocked_ezstructure` | Asserts `--seed` and `42` are passed to subprocess |
| **Integration with PLINK** | `test_full_pipeline_with_mocked_ezstructure` | Asserts `biallelic_snps_to_plink()` is called once |
| **Relevant output** (DataFrame) | `test_full_pipeline_with_mocked_ezstructure` | Asserts shape, column names, sample ID index, and row sums ≈ 1.0 |
| **Package conventions** | All tests + `admixture.py` | Uses `@doc` (numpydoc), parameter names aligned with `anopheles.py`, Mixin class for inheritance |

## Design Notes

- **Engine choice:** The class wraps `ezstructure`, a Python 3 port of fastSTRUCTURE's variational inference model. ezstructure computes expected ancestry proportions via closed-form parameter updates—no MCMC sampling required. A `random_seed` parameter (passed via `--seed`) controls initialisation of variational parameters, ensuring strict reproducibility.
- **Mixin class:** `AnophelesAdmixtureAnalysis` is designed to be inherited by `AnophelesDataResource`, following the same pattern as `AnophelesPca`, `AnophelesFstAnalysis`, etc.
- **Testing strategy & Data Access:** Due to the standard approval timelines required to obtain Google Cloud data access credentials for the MalariaGEN datasets, the test suite is built using `unittest.mock`. This isolates the class from both the Google Cloud backend and the local `ezstructure` binary. This approach guarantees that the core logic—parameter validation, PLINK file generation, subprocess execution, and DataFrame parsing—is rigorously verified without being blocked by external network or authentication constraints. It also ensures the tests are fully portable and can be run instantly by reviewers. Once authenticated access is available in a production environment, the Mixin will seamlessly retrieve data via the native `malariagen_data` API.
- **Input validation:** The method checks that sample count strictly exceeds K and raises a clear `ValueError` otherwise.
- **Output:** Returns a `pandas.DataFrame` indexed by sample ID (read from the generated `.fam` file), with columns `Ancestry_1` through `Ancestry_K`. Each row sums to 1.
