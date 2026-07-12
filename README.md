# 🛡️ Network Security Threat Detection — End-to-End ML Pipeline

An end-to-end, production-style machine learning pipeline that detects **phishing / network security threats** from URL and website-based features. Built with a modular architecture (ingestion → validation → transformation → training → deployment) instead of a single notebook, so every stage is independent, testable, and easy to debug.

---

## 🎥 Demo Videos

- [AWS S3 Bucket Setup (Model Artifact Storage)](https://youtu.be/ar9akpAy4E4)
- [AWS ECR — Docker Image Registry](https://youtu.be/41bbjl1QnkA)
- [AWS EC2 Live Deployment & FastAPI Service](https://youtu.be/M1hNGQvkJ1Y)

---

## 📌 Problem Statement

Most ML side-projects stop at "train a model in a notebook." This project goes further — simulating how a real ML system is built, tracked, and shipped:

- Ingest raw data into a live database (not static CSVs)
- Validate incoming data for schema drift before training
- Transform features consistently and reproducibly
- Train and tune models while tracking every experiment
- Package and deploy the trained model behind an API

---

## 🏗️ Architecture

```
Raw Data → MongoDB Atlas → Data Ingestion → Data Validation → Data Transformation
                                                                     │
                                                                     ▼
                                                              Model Trainer
                                                                     │
                                                                     ▼
                                                     MLflow Experiment Tracking
                                                                     │
                                                                     ▼
                                                        FastAPI Prediction Service
```

### Deployment Architecture

```
GitHub Push (main branch)
        │
        ▼
GitHub Actions — CI (lint + test)
        │
        ▼
GitHub Actions — CD (build Docker image)
        │
        ▼
   Push image to AWS ECR
        │
        ▼
Self-hosted runner on AWS EC2 pulls latest image
        │
        ▼
   Docker container runs FastAPI service
        │
        ▼
  Model artifacts pulled from AWS S3 at runtime
```

The codebase follows a clean **`constants → entity/config → components → artifacts`** structure:

| Layer | Responsibility |
|---|---|
| `constants` | Central config values (paths, DB names, schema) |
| `entity/config_entity` | Config classes (`DataIngestionConfig`, `DataValidationConfig`, `DataTransformationConfig`, `ModelTrainerConfig`, `TrainingPipelineConfig`) |
| `components` | Core pipeline logic (`DataIngestion`, `DataValidation`, `DataTransformation`, `ModelTrainer`) |
| `exception` / `logging` | Custom exception handling and centralized logging across every stage |
| `Artifacts` | Versioned outputs of each pipeline run (validated data, transformed data, trained model) |

---

## ⚙️ Pipeline Stages

1. **Data Ingestion** — Pulls raw network/phishing data from **MongoDB Atlas** into the pipeline (`push_data.py` handles loading data into Mongo).
2. **Data Validation** — Checks incoming data against an expected schema (`data_schema/`) and flags data drift before it reaches training.
3. **Data Transformation** — Cleans, encodes, and prepares features into a model-ready format, saving reusable preprocessing artifacts.
4. **Model Training** — Trains and evaluates classification models (Scikit-learn), with experiment tracking via **MLflow** (integrated with **DagsHub** for remote tracking).
5. **Serving** — Exposes predictions through a **FastAPI** service (`app.py` / `main.py`), so the model can be queried in real time.

---

## 🧰 Tech Stack

| Category | Tools |
|---|---|
| Language | Python |
| Data Store | MongoDB Atlas (`pymongo`) |
| ML | Scikit-learn, NumPy, Pandas |
| Experiment Tracking | MLflow, DagsHub |
| Serving | FastAPI, Uvicorn |
| Packaging | `setup.py`, `dill` (object serialization) |
| Containerization | Docker |
| CI/CD | GitHub Actions (self-hosted runner) |
| Cloud / Deployment | AWS S3 (model artifact storage), AWS ECR (image registry), AWS EC2 (hosting) |
| Config | `python-dotenv`, `pyaml` |

---

## 📂 Project Structure

```
NETWORK_SECURITY/
├── .github/workflows/       # CI/CD pipeline (GitHub Actions)
├── Artifacts/                # Pipeline run outputs (ingested/validated/transformed data, models)
├── Network_data/             # Raw source dataset
├── data_schema/               # Expected schema definition for validation
├── logs/                      # Pipeline execution logs
├── network_security/          # Core package
│   ├── components/            # data_ingestion.py, data_validation.py, data_transformation.py, model_trainer.py
│   ├── entity/                 # config_entity.py, artifact_entity.py
│   ├── exception/               # Custom exception handling
│   └── logging/                 # Centralized logger
├── main.py                    # Runs the full training pipeline end-to-end
├── push_data.py                # Loads raw data into MongoDB Atlas
├── Dockerfile                  # Containerization for deployment
├── requirements.txt
└── setup.py
```

---

## 🚀 Running the Project

```bash
# 1. Clone the repo
git clone https://github.com/Saksham-Arora18/NETWORK_SECURITY.git
cd NETWORK_SECURITY

# 2. Install dependencies
pip install -r requirements.txt

# 3. Set up environment variables (.env)
MONGO_DB_URL=<your-mongodb-atlas-connection-string>

# 4. Push raw data to MongoDB (one-time)
python push_data.py

# 5. Run the full training pipeline
python main.py
```

---

## 📸 Screenshots

All screenshots are available in the [`screenshots/`](./screenshots) folder.

| Screenshot | Description |
|---|---|
| `github-actions-pipeline.png` | CI/CD pipeline — Integration, Build & Push, and Deployment stages all passing |
| `ecr-repo.png` | Docker image pushed to AWS ECR |
| `ec2-deployment.png` | Application running live on the EC2 instance |
| `fastapi-docs.png` | FastAPI Swagger UI (`/docs`) served from the deployed instance |

---

## 📈 Key Learnings

- Structuring an ML project as decoupled components (rather than one script) makes debugging and re-running individual stages far easier.
- Data validation before training catches silent data-quality issues that would otherwise corrupt a model without any error being raised.
- Experiment tracking (MLflow + DagsHub) removes the guesswork of "which run/config actually performed best."
- Wrapping the trained model behind a FastAPI service is what actually makes an ML pipeline usable, not just trainable.

---

## 👤 Author

**Saksham Arora**
🔗 [GitHub](https://github.com/Saksham-Arora18) · [LinkedIn](https://www.linkedin.com/in/saksham-arora-377b47335)