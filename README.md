# FlyRank ML Engineering Internship Portfolio

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![ML Track](https://img.shields.io/badge/Focus-Applied%20ML%20%26%20Ranking%20Systems-00C7B7.svg)]()
[![Environment](https://img.shields.io/badge/IDE-VS%20Code%20Local-705697.svg)]()

## About the Author
**Muhammad Muzammil** *Undergraduate Software & AI Engineering Student | Applied Machine Learning Engineer*

I am an engineering student specializing in autonomous workflows, backend API integrations, and Retrieval-Augmented Generation (RAG) architectures. This repository documents my industry progression through the 12-week **FlyRank Machine Learning Track**, where I build production-grade information retrieval and ranking systems.

### Connect with Me
* **LinkedIn:** [linkedin.com/in/m-muzammil-](https://www.linkedin.com/in/m-muzammil-/)
* **GitHub:** [github.com/muzammilsharf](https://github.com/muzammilsharf)
* **Email:** [sharfmuzamil@gmail.com](mailto:your-email@example.com)

---

## Executive Summary
This repository serves as my active development workspace, weekly assignment binder, and technical portfolio for the **FlyRank Machine Learning Engineering Track**. 

Unlike academic data science repositories that focus solely on static model training, this project emphasizes **production engineering standards**: writing reproducible automation scripts, handling large-scale data streaming without out-of-memory errors, analyzing real search engine optimization (SEO) signals, and developing interpretable, highly accurate ranking algorithms.

---

## Technical Stack & Tooling
* **Core Languages:** Python, Bash / Zsh
* **Data Science & Math:** `pandas`, `numpy`, `scikit-learn`
* **Data Engineering & Storage:** `duckdb` (Out-of-core SQL execution), Parquet formatting
* **ML Platforms & Pipelines:** Hugging Face Hub (`hf` CLI), Custom Ranking Architectures
* **Development Environment:** VS Code (Local Linux environment utilizing isolated `venv` virtual environments), Google Colab, Jupyter Notebook

---

## Repository Architecture
```text
flyrank-ml-internship/
├── work/                  # MY WORKSPACE: Weekly assignment implementations & Capstone
│   └── notebooks/         # Executed interactive analysis & model iterations
├── scripts/               # REFERENCE PIPELINE: Immutable end-to-end automation scripts
├── data/                  # DATASETS: Local git-safe raw and processed feature vectors
├── outputs/               # BENCHMARKS: Generated charts, queues, and model metrics
└── SETUP.md / GUIDE.md    # Documentation and architecture specifications
```
---

## Weekly Engineering & Discovery Log
* Week 1: Pipeline Automation & Baseline Ranking Discovery

   - Objective: Establish a locally reproducible ML pipeline and evaluate the performance difference between manual heuristic rules and learned statistical models on SEO content refresh data (30,000+ records).

   - Key Technical Discoveries:

      Heuristics vs. Machine Learning: Built and evaluated a transparent handwritten baseline rule (Precision@50 of ~0.24). Successfully trained a Random Forest model that outperformed the manual heuristic by roughly 3x (Precision@50 of ~0.74), proving the necessity of learned multi-variable feature weighting in search ranking.

      Data Leakage & Tree Ceilings: Conducted depth experiments on decision trees using a highly correlated "leaky" feature (trend_pct). Demonstrated that once a tree achieves pure leaf nodes at a shallow depth (max_depth=2), raising the tree ceiling to max_depth=4 yields identical Top-K ranking accuracy because pure branches short-circuit further splitting.

      Top-K Ranking Optimization: Analyzed why standard classification accuracy is a flawed metric for information retrieval systems, shifting focus toward optimizing Precision@50 to evaluate only the highest-confidence queue recommendations.

* Week 2: Interpretable Models & Feature Engineering

   - Status: In Progress...

   - Objective: Exploring linear formulations, decision boundaries, and mathematical feature transformations to improve ranking explainability for stakeholders.

* Week 3: Gated Data Streaming & Large-Scale Pipelines

   - Status: Upcoming...

   - Objective: Integrating FlyRank warehouse datasets (79M+ rows) utilizing out-of-core DuckDB streaming and Hugging Face token authentication inside local hardware RAM constraints.

---

## Local Setup & Reproduction Guide

To clone this repository and reproduce the automated ML pipeline locally:


### 1. Clone the repository
```
git clone [https://github.com/muzammilsharf/flyrank-ml-internship.git](https://github.com/muzammilsharf/flyrank-ml-internship.git)
cd flyrank-ml-internship
```

### 2. Initialize and activate a local virtual environment
```
python -m venv venv
source venv/bin/activate
```

### 3. Install required dependencies
```
pip install --upgrade pip
pip install -r requirements.txt
```

### 4. Authenticate Hugging Face CLI (Required for streaming warehouse datasets)
```
hf auth login
```

### 5. Execute the master automated pipeline
```
python scripts/run_all.py
```