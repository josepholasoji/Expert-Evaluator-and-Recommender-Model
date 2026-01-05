[![Build Status](https://travis-ci.org/josepholasoji/expert-valuator.svg?branch=master)](https://travis-ci.org/josepholasoji/expert-valuator)
![GitHub](https://img.shields.io/github/license/josepholasoji/expert-valuator?style=social)
[![DeepScan grade](https://deepscan.io/api/teams/5816/projects/7645/branches/80659/badge/grade.svg)](https://deepscan.io/dashboard#view=project&tid=5816&pid=7645&bid=80659)

# Expert-Evaluator-and-Recommender-Model

A machine learning service to detect an expertice of an individual based on a project details and readme files

This codebase contains repo code for:

1) Collecting developer/project signals from GitHub via the GraphQL API.
2) Generating a synthetic training dataset from “truth templates”.
3) Training a Random Forest classifier (TensorFlow 1.x `tensor_forest`) to categorize expertise.

NB: This repo is not packaged as a single runnable service; it’s a set of scripts/notebook-work that demonstrate the pipeline pieces.

## Repository layout

- `expert_recommender/`
	- `queries.py`: Example GitHub GraphQL query that pulls user + repo metadata and commit history.
	- `functions.py`: Minimal GraphQL client (`run_query`).
	- `project_work_graphql_tensorflow.py`: Notebook-style scratchpad (JSON). Not a normal `.py` script.
- `model_generator/`
	- `definitions.py`: “Truth template” definitions and a dataset expansion helper (`gen_classification_data`).
	- `gen.py`: Builds `truth_template_dataset.csv` from templates.
	- `random_forest_trainer.py`: Trains/tests a TF1 Random Forest on the generated CSV.
	- `truth_templates.py`: Legacy/unfinished template draft (not used by training).

## Prerequisites

- Python 3.6–3.8 (the training code uses TensorFlow 1.x APIs)
- A GitHub Personal Access Token for GraphQL calls (Classic PAT works). Store it in an env var:
	- Windows PowerShell: `setx GITHUB_TOKEN "<YOUR_TOKEN>"`
	- macOS/Linux: `export GITHUB_TOKEN="<YOUR_TOKEN>"`

Python packages used by the scripts:

- `requests`
- `pandas`
- `numpy`
- `scikit-learn`
- `tensorflow==1.*` (the trainer uses `tensorflow.contrib`, removed in TF2)

## Quick start

### 1) (Optional) Query GitHub GraphQL

From the repo root:

- `python -c "import expert_recommender.queries as q; from expert_recommender.functions import run_query; print(run_query(q.developer_query))"`

Notes:

- Update the query in `expert_recommender/queries.py` (login, emails, number of repos) to match what you want to fetch.
- `expert_recommender/functions.py` reads `GITHUB_TOKEN` at runtime.

### 2) Generate the synthetic dataset

Run from the `model_generator` folder (imports are written for that working directory):

- `cd model_generator`
- `python gen.py`

This writes `truth_template_dataset.csv` into the current directory.

### 3) Train/test the Random Forest (TensorFlow 1.x)

Also from `model_generator`:

- `python random_forest_trainer.py`

You should see training loss/accuracy prints and a final `Test Accuracy:` line.

## How the synthetic data works

The dataset generator expands template ranges from `model_generator/definitions.py` into rows with these features:

- `language` (tokenized)
- `library` (tokenized)
- `imported_library` (0/1)
- `commit_count`
- `days_since_last_commit`
- `code_churn`

Target label:

- `classification` (tokenized): `disqualified`, `novice`, `intermediate`, `expert`

## Known limitations

- Training uses TF1 `tensorflow.contrib.tensor_forest`, which does not run on TensorFlow 2.x.
- The GitHub data pull and the ML model are not currently wired together (no end-to-end “score a developer” script).
- `expert_recommender/project_work_graphql_tensorflow.py` is notebook JSON checked in with a `.py` extension.

## License

MIT (see `LICENSE`).
