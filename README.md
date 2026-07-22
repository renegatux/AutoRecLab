# Team 6 — AutoRecLab in the Wild

This repository contains the logs, configuration, and reproduction notes for Team 6's Machine Learning Lab project:

**AutoRecLab in the Wild: Prompt Sensitivity Meets Run Stability**

We investigated how sensitive and stable AutoRecLab is when running recommender-system experiments with prompts of different detail levels. Our study focused on three prompt variants, two LLM backends, and 16 experimental runs in total.

The final report and presentation were submitted separately via the course submission system and are not included in this repository.

---

## Repository Information

**Fork:** <https://github.com/renegatux/AutoRecLab>

**Branch / commit used for the project:**

```text
Branch: main / develop
Commit: cf2be33f70843aa9ac9aa9b0fcb4eb543185469b
```

The experimental runs were executed on the `main` branch. Later upstream contributions such as pull requests, issues, and the discussion were prepared for the upstream project and/or the `develop` workflow.

The experiment logs are available in:

```text
logs/output_log/
```

---

## Upstream Contributions

As part of the project, we also contributed to the upstream AutoRecLab repository.

### Pull Requests

- [#54 Dead-code removal and redundant good-nodes list cleanup](https://github.com/ISG-Siegen/AutoRecLab/pull/54)
- [#63 Parallel per-requirement scoring](https://github.com/ISG-Siegen/AutoRecLab/pull/63)
- [#64 Interpreter.cleanup_session stub implementation](https://github.com/ISG-Siegen/AutoRecLab/pull/64)

### Issues

- [#47 Pre-run file validation before tree search](https://github.com/ISG-Siegen/AutoRecLab/issues/47)
- [#62 Reviewer scores miss scientific invalidity](https://github.com/ISG-Siegen/AutoRecLab/issues/62)

### Discussion

- [#61 Estimated time remaining during runs](https://github.com/ISG-Siegen/AutoRecLab/discussions/61)

---

## Repository Structure

The most relevant project files are:

```text
logs/output_log/       Raw logs of our AutoRecLab runs
treesearch/            AutoRecLab tree-search and agent implementation
config.toml            Main configuration file, including model selection
main.py                AutoRecLab entry point
README.md              This project README
```

The raw logs are stored in:

```text
logs/output_log/
```

The repository currently contains the following experiment log files:

```text
Detailed_prompt_Run1.txt
Detailed_prompt_Run2.txt
Detailed_prompt_Run3.txt
Detailed_prompt_Run4.txt

Preprint_prompt_GPT_5.4_Run1+Run2.rtf
Preprint_prompt_GPT_5.4_Run3.txt
Preprint_prompt_GPT_5.4_Run4.txt

Preprint_prompt_GPT_Nano_Run1.rtf
Preprint_prompt_GPT_Nano_Run2.rtf
Preprint_prompt_GPT_Nano_Run3.txt
Preprint_prompt_GPT_Nano_Run4.txt

Simple_promt_Run1.rtf
Simple_promt_Run2.rtf
Simple_promt_Run3.rtf
Simple_promt_Run4.rtf
```

---

## Research Question

Our main research question was:

> How sensitive and how stable is AutoRecLab when the same recommender-systems task is given with different levels of prompt detail?

We tested whether more detailed prompts improve AutoRecLab's formal success rate and whether repeated runs with the same prompt and model produce stable results.

---

## Prompt Variants

We used three prompt variants with increasing levels of detail.

### 1. Simple Prompt

```text
Implement a simple collaborative filtering recommender system using the MovieLens dataset.
```

This prompt leaves most design decisions to AutoRecLab, including algorithm choice, dataset handling, and evaluation metrics.

### 2. Detailed Prompt

```text
Implement a collaborative filtering recommender using matrix factorization.
Load data from movielens.csv in the working directory.
Evaluate with RMSE and MAE metrics.
```

This prompt explicitly defines the algorithm family, the dataset filename, and the evaluation metrics.

### 3. Preprint Prompt

```text
I’d like to run an experiment to quantify how much data split random seeds affect recommender system accuracy. Please use LensKit 0.14.4 to test three algorithms: ALS, ItemKNN, and Pop. Run this on the following three datasets with implicit feedback: MovieLens100K, Amazon Video Games, Last.FM. The raw files are stored in your working directory with the filenames u.data, VideoGames.csv, UserTaggedArtists-timestamps.dat. First, preprocess all datasets with 5-core filtering. For the Amazon and MovieLens datasets, please also convert any ratings greater than 3 to implicit interactions. Here’s the main experimental procedure: Generate 5 different random seeds for data splitting. For each algorithm, dataset, and seed, please do a user-based 80/20 holdout split. Train all models using standard hyperparameters. For the analysis, I need you to measure nDCG@k and Precision@k for k=1, 5, 10 and conduct a short statistical analysis.
```

This prompt is the most complex one. It requests a multi-dataset, multi-algorithm experiment with five random seeds and ranking metrics.

---

## Experimental Setup

We ran AutoRecLab locally via **UV**, not Docker.

Experiments were conducted on two machines:

- MacBook Air with Apple Silicon, macOS
- Windows desktop

We used two LLM backends:

- `gpt-5-nano`
- `gpt-5.4`

The model was changed manually in `config.toml`.

---

## Model Configuration

The LLM backend used by AutoRecLab can be changed in:

```text
config.toml
```

Relevant section:

```toml
[agent.code]
model = "gpt-5-nano"
model_temp = 1.0
```

For our experiments, we used `gpt-5-nano` for the Simple and Detailed prompts and both `gpt-5-nano` and `gpt-5.4` for the Preprint prompt.

Some model names from the default or attempted configuration were not available or not compatible with our course OpenAI project setup. Therefore, the model name had to be adjusted manually.

---

## Dataset Placement

The raw datasets are not included in this repository due to size. They must be downloaded separately and placed into the expected workspace directory before running the experiments.

For our local UV setup, AutoRecLab expected dataset files in:

```text
workspace/working/
```

For the Simple and Detailed prompts, the expected file was:

```text
workspace/working/movielens.csv
```

For the Preprint prompt, the expected files were:

```text
workspace/working/u.data
workspace/working/VideoGames.csv
workspace/working/UserTaggedArtists-timestamps.dat
```

During our experiments, dataset placement was one of the recurring sources of errors. The original instructions and the generated code were not always consistent regarding whether files should be placed in `workspace/` or `workspace/working/`.

---

## Reproducing the Experiments

### Windows PowerShell

The minimal command sequence used for our Windows-based runs was:

```powershell
uv sync
$env:OPENAI_API_KEY="<your-key>"
mkdir out
mkdir workspace
uv run main.py
```

If the folders already exist, the shorter version is sufficient:

```powershell
uv sync
$env:OPENAI_API_KEY="<your-key>"
uv run main.py
```

For a cleaner reproduction setup with datasets, we recommend ensuring that the dataset directory exists:

```powershell
mkdir out
mkdir workspace
mkdir workspace\working
```

Then place the required dataset files into:

```text
workspace/working/
```

Start AutoRecLab:

```powershell
uv run main.py
```

Paste one of the prompts listed above and end the input with:

```text
!start
```

### macOS / Linux

```bash
uv sync
export OPENAI_API_KEY="<your-key>"
mkdir -p out workspace/working
uv run main.py
```

Then paste one of the prompts and end the input with:

```text
!start
```

---

## Logs and Score Extraction

AutoRecLab assigns an internal reviewer score to generated candidate nodes. In the logs, these scores appear in lines of the form:

```text
NodeScore(score=92.0, feedback=..., is_satisfactory=True)
```

The score is AutoRecLab's internal reviewer rating from 0 to 100. It reflects how well the generated code satisfies the formal requirements generated by AutoRecLab.

However, this score is not the same as scientific validity. A high score can still occur when the run has scientific problems such as overfitting, missing metrics, synthetic data fallback, or algorithm mismatch.

The raw logs can be found in:

```text
logs/output_log/
```

---

## Results Summary

The following table summarizes the 16 experimental runs.

| Series | Model | Runs | Satisfactory | Scientifically Valid |
|---|---|---:|---:|---:|
| Simple Prompt | GPT-5 Nano | 4 | 1 | 1 |
| Detailed Prompt | GPT-5 Nano | 4 | 3 | 1 |
| Preprint Prompt | GPT-5 Nano | 4 | 0 | 0 |
| Preprint Prompt | GPT-5.4 | 4 | 0 | 0 |
| **Total** | — | **16** | **4** | **2** |

Main observations:

- More prompt detail improved the formal success rate.
- Formal success did not imply scientific validity.
- The Preprint prompt failed in all 8 runs.
- The internal AutoRecLab score mainly reflected code structure and requirement coverage, not full scientific correctness.

---

## Scientific Validity Assessment

In addition to AutoRecLab's internal score, we manually assessed whether each run was scientifically valid.

A run was counted as valid only if all of the following criteria were satisfied:

1. real data were used, not synthetic fallback data;
2. the implemented algorithm matched the prompt;
3. the required metrics were actually reported;
4. there was no catastrophic overfitting or obvious data leakage.

This manual assessment explains why a run with a high AutoRecLab score could still be considered invalid.

For example, a run could receive a score of 92/100 because the code executed and matched many formal requirements, but still be scientifically invalid if validation error diverged while training error collapsed.

---

## Known Issues Encountered

During reproduction, we encountered several practical issues:

- The `out/` directory was missing on first launch and had to be created manually.
- Some model names from the original/default configuration were not available in the course OpenAI project.
- Dataset path handling was inconsistent between `workspace/` and `workspace/working/`.
- Some generated code failed because `matplotlib` was missing, even though plotting was not essential for the experiment.
- The Preprint prompt repeatedly triggered LensKit 0.14.4 API incompatibilities.
- Some runs crashed with `IndexError` when no valid tree-search node was found.
- On macOS, LensKit-related multiprocessing caused additional stability problems in some runs.
- The reviewer score did not reliably detect scientific issues such as overfitting or missing final metrics.

These issues motivated our proposed improvements: pre-run validation, more robust error handling, optional plotting, and scientific-validity checks in the reviewer.

---

## Citation / Project Reference

Project title:

```text
AutoRecLab in the Wild: Prompt Sensitivity Meets Run Stability
```

Team:

```text
Team 6
Ksenia Khokhlova
Artem Dneprovskii
Machine Learning Lab
University of Siegen
```
