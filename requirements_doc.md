# FlowAI
FlowAI — Requirements Doc (v1)

Project: FlowAI (MVP — prompt → model)
Start date: 2025-11-13
Target MVP completion: 2026-03-15
Owner: Vedanth Yada

1. Project Summary

FlowAI is a modular, prompt-driven system that takes a natural-language or CLI prompt such as “Train a model to predict X from this CSV” and executes an end-to-end ML workflow: data ingestion → cleaning/validation → model training & tuning → evaluation → model artifact + serving endpoint + report. The MVP supports tabular and text classification/regression tasks, runs locally (single-machine), and logs runs in MLflow (file backend).

2. MVP Scope (in/out)

Included

Tasks: Tabular classification & regression; text classification.

Data sources: local file upload (CSV, XLSX), dataset by URL.

Automation: preprocessing (missing values, basic encoders), model selection, hyperparameter tuning, evaluation reporting.

Outputs: cleaned dataset snapshot, trained model artifact, JSON + HTML evaluation report, FastAPI inference endpoint, MLflow run logs.

Interface: CLI + notebook; prompt-driven CLI (text command).

Dev infra: local filesystem, venv or Docker, MLflow with file backend.

Excluded (MVP)

Production-grade multi-node deployment, real-time streaming ingestion, large-scale distributed training, complex web scraping, automated labelling, GPU-dependent deep models (beyond small local PyTorch runs), human-in-the-loop labeling UI.

3. User Stories

As a user, I can run flowai run --prompt "Train a classifier on data/raw/housing.csv" and receive a model + evaluation report.

As a user, I can upload a CSV and choose to run the default pipeline from a notebook or CLI.

As a developer, I can add a new data connector (e.g., Google Sheets) without changing training code.

As a user, I can query the model via a simple REST endpoint /predict.

As a user, I can inspect MLflow to review a run’s parameters, metrics, and artifacts.

4. Acceptance Criteria (concrete)

AC1: A CLI command or single text prompt triggers a full pipeline run (ingest → clean → train → eval → save) end-to-end.

AC2: The pipeline logs every run to MLflow (params, metrics, model artifact).

AC3: For tabular classification on at least one toy dataset (e.g., UCI/CSV provided), model training completes in < 15 minutes on a dev laptop and saves a model file in models/.

AC4: FastAPI app serves a /predict endpoint that returns predictions for JSON input and loads the latest model.

AC5: Repo contains requirements.txt, README with quickstart, and basic architecture diagram (mermaid or png) in docs/.

AC6: Modular code structure present (connectors/, etl/, train/, serve/, prompt_parser/).

AC7: Basic unit tests for core modules (CI runs lint + tests).

5. Success Metrics

Functional: At least one successful end-to-end run (tabular) and one (text) during acceptance testing.

Reliability: 90% of runs complete without developer intervention during demo runs (5/5).

Performance: End-to-end runtime for toy dataset < 15 minutes on dev machine.

Usability: User can trigger a run with a single CLI line or notebook cell.

Traceability: Every run logged to MLflow with retrievable artifacts.

6. High-level Architecture (components)

Prompt Parser — parses user prompt into structured pipeline spec.

Pipeline Orchestrator — maps spec → sequence of steps and executes.

Data Connectors — CSV, URL, (future: Google Sheets, REST API).

ETL / Data Validation — cleaning, missing-value handling, simple schema checks.

Feature Engineering — automatic encoders, tokenization for text.

Trainer / Auto-ML — scikit-learn pipelines, hyperparameter search (Optuna or scikit-learn + GridSearch).

Model Registry — MLflow (file backend).

Serving — FastAPI for inference.

Monitoring (MVP minimal) — basic logs + saved eval report + future drift detection hooks.

Mermaid (add to docs):

flowchart LR
  A[User prompt / CLI] --> B[Prompt Parser]
  B --> C[Orchestrator]
  C --> D[Data Connectors]
  D --> E[ETL & Validation]
  E --> F[Feature Engineering]
  F --> G[Trainer & Tuning]
  G --> H[MLflow Model Registry]
  H --> I[Serving API]
  I --> J[Log & Reports]

7. Tech Stack (MVP)

Language: Python 3.11+

Core libs: pandas, scikit-learn, transformers (optional), PyTorch (optional), nltk/spacy (optional for text), Optuna (optional)

Orchestration: simple orchestrator scripts / Prefect (optional MVP)

Model tracking: MLflow (file backend)

Serving: FastAPI + Uvicorn

Packaging: venv / Docker (docker-compose optional)

Storage: local filesystem (data/, models/); S3/GCS planned for prod

CI: GitHub Actions (lint + pytest)

8. Data & Privacy

All dev data stored locally in data/ with data/raw/ and data/processed/.

Sensitive data: do not commit real PII into repo. Provide .env + .gitignore rules.

For demos, use or synthesize non-sensitive toy datasets.

9. Milestones & Timeline (high level)

Phase 0 (Nov 13–Nov 30): Requirements doc (this), repo + env, toy pipeline smoke test.

Phase 1 (Dec 1–Dec 31): Data connectors, ETL, schema & validation, dataset snapshots.

Phase 2 (Jan 1–Feb 15): Trainer automation, model selection, hyperparameter tuning, MLflow logging.

Phase 3 (Feb 16–Mar 15): Serving, prompt-driven runner, docs, tests, final demo.

(Weekly subtasks to be tracked in project board.)

10. Deliverables

requirements_doc.md (this file) — committed.

Repo scaffold + requirements.txt + .gitignore, README.

Minimal ETL script + CSV connector.

train_toy.py — trains+stores a model and logs to MLflow.

FastAPI serve/app.py with /predict.

MLflow runs of at least two successful example datasets.

README with Quickstart and Demo instructions.

Basic unit tests and CI pipeline.

11. Risks & Mitigation

Risk: Data quality prevents automation. → Mitigation: implement strict validation, fail-fast with clear error messages and sample data check tool.

Risk: Over-ambitious feature set delays MVP. → Mitigation: adopt narrow MVP scope (tabular + text only).

Risk: Long model training times on local machine. → Mitigation: use small toy datasets for dev, add early-stopping & subsampling.

Risk: Time constraints. → Mitigation: weekly checkpoints and prioritized backlog.

12. Open Questions (to resolve during Phase 0)

CLI syntax and prompt grammar: exact format for initial MVP (define template).

Which two toy datasets to standardize on for acceptance tests? (Suggest: UCI/CSV for tabular; IMDb small or SST subset for text).

Will you use Optuna or scikit-learn grid search for HPO initially?

CI level: what tests must pass before merge? (unit tests + lint).
