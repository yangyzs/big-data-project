# Efficient Semantic Joins Using Traditional Matching and LLMs

**Team:** Three of a Kind  
**Task:** Semantic joining/entity matching on the Auto-FuzzyJoin benchmark.

## Overview

We evaluate string-similarity matching, embedding-based candidate retrieval, Qwen verification, and two hybrid methods on BasketballTeam, Country, and Device. We evaluate precision, recall, F1, LLM verification calls, and recorded LLM inference time.

## Pipeline

1. **Traditional baseline:** RapidFuzz `fuzz.ratio` after `lower().strip()` with threshold 80.
2. **Candidate retrieval:** `sentence-transformers/all-MiniLM-L6-v2`, normalized embeddings and cosine similarity; top 5 left-side candidates for each right-side value.
3. **Qwen-only baseline:** Verify every top-5 pair with `Qwen/Qwen2.5-1.5B-Instruct` using the generic YES/NO/UNSURE prompt.
4. **Fixed-candidate hybrid:** Automatically accept threshold-80 fuzzy matches *within* top-5; verify the remaining top-5 pairs with Qwen.
5. **Full hybrid:** Automatically accept all threshold-80 fuzzy matches (including those outside top-5) and verify the remaining top-5 pairs with Qwen.

Only `YES` is treated as an accepted LLM match. `NO`, `UNSURE`, and invalid outputs/abstentions are not accepted. Matching is evaluated against benchmark ID pairs.

## Datasets

Auto-FuzzyJoin benchmark repository: https://github.com/chu-data-lab/AutomaticFuzzyJoin  
Benchmark directories: `src/autofj/benchmark/BasketballTeam`, `Country`, and `Device`.

| Dataset | Left | Right | Ground-truth pairs |
|---|---:|---:|---:|
| BasketballTeam | 928 | 166 | 166 |
| Country | 2,791 | 291 | 291 |
| Device | 6,933 | 658 | 658 |

The notebook fetches public benchmark CSV files from the upstream repository. We do not duplicate the benchmark here.

## Results

Full evaluation results: [`results/cross_dataset_summary.csv`](results/cross_dataset_summary.csv).

| Dataset | Qwen-only F1 | Fixed-hybrid F1 | Reduction in LLM calls |
|---|---:|---:|---:|
| BasketballTeam | 0.5633 | 0.5910 | 9.04% |
| Country | 0.4242 | 0.4152 | 14.09% |
| Device | 0.3604 | 0.3575 | 26.60% |

The main finding is that the hybrid reduces LLM calls across all three datasets, but F1 gains are dataset-dependent. Accepting fuzzy matches outside the embedding candidate pool introduces many false positives on Country and Device.

### Inference time

See [`results/runtime_comparison.csv`](results/runtime_comparison.csv). Hybrid inference time was **estimated by summing logged per-pair times** for pairs retained in the hybrid queue, rather than rerunning the full hybrid pipeline. This is **not an end-to-end runtime benchmark**; preprocessing, embedding, candidate generation, model loading, and I/O must be included for full pipeline comparisons.

## Reproducing the experiments

1. Open your cleaned combined notebook from [`notebooks/`](notebooks/) in Google Colab. **Add the final working notebook there** (it is not included in this starter package).
2. Select an appropriate GPU runtime.
3. Install the dependencies **once**, verify versions, and restart the session if required. Avoid cells that uninstall Transformers or run `pip install -U transformers` later in the notebook.
4. Run the notebook in order, keeping dataset-specific variable names and unique output files.
5. Existing saved predictions can be loaded from Drive for evaluation without rerunning Qwen. Save output CSVs using descriptive dataset-specific filenames.

This starter package contains the final results summary, documentation, and environment specification. The working Colab notebook and raw prediction CSV files still need to be copied from the project's Drive folder.

## Important caveats

- BasketballTeam was used for exploratory threshold analysis. The threshold-80 configuration should be treated as development-selected when interpreting results on the other datasets.
- Not every experiment was timed in a single controlled end-to-end run.
- Some Qwen responses were invalid (e.g., `SURE`) and were treated as abstentions, not silently converted into a match.
- Per-pair inference times were recorded across notebook sessions; environment/hardware details and package versions should be documented for reproducibility.
- The Qwen-only baseline uses top-5 candidates, whereas the **full hybrid** can add fuzzy matches outside top-5. These are different candidate coverage strategies.

## Repository structure

```text
notebooks/       Clean combined Colab notebook (add it here)
results/         Cross-dataset metrics and recorded inference times
docs/            Experiment notes, methodological caveats
requirements.txt
.gitignore
README.md
```
