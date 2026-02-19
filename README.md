# Production ML DVC Pipeline 📦

A reproducible **MLOps pipeline** for building a spam detection model using **DVC**, **Git**, and modular Python scripts. The project demonstrates end-to-end workflow from data ingestion through model evaluation with experiment tracking via `dvclive` and parameter configuration in `params.yaml`.

---

## 🚀 Project Overview

This repository contains the code and configuration needed to ingest raw spam data, preprocess and engineer features, train a machine learning model, evaluate its performance, and track experiments. It’s designed to be portable and versioned: data, models, and metrics are managed by DVC while code and configuration live under Git.

Key elements:

- **Data ingestion**: download and clean the spam dataset. 
- **Feature engineering**: transform text into TF–IDF vectors.
- **Model building**: train a random forest classifier and serialize the model.
- **Model evaluation**: compute metrics, log them with `dvclive`, and save reports.
- **Experiment management**: run parameterized experiments via DVC experiments.

The pipeline is orchestrated through `dvc.yaml` (with optional parameters in `params.yaml`) and can be reconstructed with `dvc repro` or `dvc exp run`.

---

## 🗂 Repository Structure

```
├── dvc.yaml            # DVC pipeline stages
├── params.yaml         # Parameter configuration for stages
├── src/                # Modular pipeline scripts
│   ├── data_ingestion.py
│   ├── feature_engineering.py
│   ├── model_building.py
│   ├── model_evaluation.py
├── data/               # (auto-managed by DVC) raw/interim/processed
├── models/             # trained model artifact(s)
├── reports/            # evaluation metrics, plots, etc.
├── experiments/        # notebook, raw data, experiment artifacts
├── dvclive/            # dvclive experiment logs
├── logs/               # pipeline logging
├── .gitignore          
├── .dvcignore          
├── LICENSE
└── README.md           # (this file)
```

---

## 🛠️ Setup Instructions

1. **Clone the repository** (or start a new GitHub repo):
   ```bash
   git clone <repo-url> production-ml-dvc-pipeline-
   cd production-ml-dvc-pipeline-
   ```

2. **Create a Python environment** (recommended: `venv` or Anaconda):
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```
   *(or install manually: `numpy pandas scikit-learn dvc dvclive nltk wordcloud`)*

3. **Install DVC**: 
   ```bash
   pip install dvc
   dvc init          # initialize DVC in the repo (already done)
   ```

4. **Download NLTK data** used by the pipeline:
   ```python
   import nltk
   nltk.download('stopwords')
   nltk.download('punkt')
   nltk.download('punkt_tab')
   ```

5. **Configure parameters** in `params.yaml` (default values exist). Example:
   ```yaml
   data_ingestion:
     test_size: 0.2
   feature_engineering:
     max_features: 5000
   model_building:
     n_estimators: 100
     random_state: 42
   ```

6. **Optional: set up a cloud remote (AWS S3)**
   ```bash
   # configure AWS credentials first
   aws configure
   # add and use an S3 bucket as the default DVC remote
   dvc remote add -d dvcpipeline(or else) s3://your_s3_bucket
   dvc remote modify dvcpipeline region us-east-1  # or your region
   ```
   You can inspect the remote at any time with `dvc remote list` or by opening
   `.dvc/config` (the same file is referenced in `project_workflow_dvc_mlops.txt`).
   After pushing data/models they will be stored in the specified S3 bucket.

7. **Run the DVC pipeline**:
   ```bash
   dvc repro      # executes all stages according to dvc.yaml
   ```

8. **Experiment with parameters**:
   ```bash
   dvc exp run                 # creates an experiment and logs via dvclive
   dvc exp show                # list experiments
   dvc exp apply <exp-name>    # apply previous experiment
   ```
   Modify `params.yaml` and re-run to generate new results.

8. **Track results**: generated metrics appear in `reports/metrics.json` and `dvclive/`.

9. **Commit and push**:
   ```bash
   git add .
   git commit -m "Updated pipeline"
   git push origin main
   dvc push    # push data and models to remote storage if configured
   ```

---

## 🧩 Pipeline Components

Each script in `src/` can be run standalone or via the DVC pipeline. They all load configuration from `params.yaml` and write logs to `logs/`.

- **`data_ingestion.py`** – loads CSV from the internet, cleans columns, splits into train/test and saves to `data/raw/`.
- **`feature_engineering.py`** – reads the split data, vectorizes text using TF–IDF (controlled via `max_features`), and writes processed CSVs.
- **`model_building.py`** – trains a `RandomForestClassifier` with hyperparameters from `params.yaml` and serializes the model to `models/model.pkl`.
- **`model_evaluation.py`** – loads the test set and model, computes accuracy/precision/recall/AUC, logs them with `dvclive`, and writes a JSON report.

Helper functions in each module manage parameter loading (`load_params`) with logging and error handling.

---

## 📦 DVC & Experiment Tracking

- `dvc.yaml` defines the sequence of stages (ingest → featurize → train → evaluate).
- `params.yaml` allows you to vary dataset splits, vectorizer size, and model hyperparameters.
- `dvclive` within `model_evaluation.py` captures metrics and parameters automatically when `dvc exp run` is executed.

Example of running an experiment:

```bash
# tweak params then run:
dvc exp run
# view results
dvc exp show
# optionally apply or remove
```

Logs and plots for each experiment are stored under `dvclive/` and can be visualized with the [DVC extension for VSCode](https://marketplace.visualstudio.com/items?itemName=iterative.dvc).

---

## 📝 Notes & Tips

- Ensure `.gitignore` and `.dvcignore` exclude large/auto-generated files such as `data/`, `models/`, and `dvclive/`.
- Update `requirements.txt` when adding new dependencies.
- Use `dvc repro` for a full pipeline run, `dvc exp run` for parameter experiments.
- The `experiments/my_notebook.ipynb` notebook contains exploratory work and can be used for quick prototyping.
- Logs are written to `logs/<component>.log` for debugging.

---

## 📄 License

This project is licensed under the terms of the [LICENSE](./LICENSE) file.

---

Happy MLOps! 👩‍💻👨‍💻

Feel free to adapt, extend or contribute. Questions or ideas? Open an issue or pull request!