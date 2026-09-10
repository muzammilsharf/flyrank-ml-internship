# Search Traffic Decay Prediction — FlyRank ML Internship Capstone

I built a machine learning model that predicts which web pages are likely to lose search traffic, so content teams know which pages to review first. This repo has the full project: feature pipeline, model, validation, and a deployed research paper.

**Read the full paper:** https://muzammilsharf.github.io/flyrank-ml-internship/

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![ML Track](https://img.shields.io/badge/Focus-Applied%20ML%20%26%20Ranking%20Systems-00C7B7.svg)]()
[![Environment](https://img.shields.io/badge/IDE-VS%20Code%20Local-705697.svg)]()

## What this project does

Content teams manage thousands of pages and cannot check every one every week. This model ranks pages by risk of decline, using real search and analytics data, so editors can focus their limited time on the pages that need it most.

## Key result

| Method | Precision@50 |
|---|---|
| Simple rule-based baseline | 24.0% |
| **Final model (LightGBM)** | **78.8%** (average across 5 tests) |

The model correctly identifies declining pages in its top recommendations about 3.9x better than random guessing, and more than 3x better than the baseline rule.

## What I actually did

- Accessed FlyRank's search analytics warehouse (78 million rows of Google Search Console and Google Analytics data, pre-built and hosted by FlyRank) and built a feature-engineering pipeline on top of it using DuckDB and Python, extracting 90-day rolling traffic windows and content signals under limited local hardware.
- Designed the prediction label (30-day forward decline vs. trailing traffic) and the full feature set, including content metadata joined from a separate reference table.
- Found and fixed three real data problems before trusting any result: a filter that silently dropped most of the usable data, a feature that leaked future information into the model, and a feature that was secretly tied to the calendar month instead of real page age.
- Compared four model types (Logistic Regression, Decision Tree, Random Forest, LightGBM) under a validation design built to prevent the model from learning shortcuts instead of real patterns.
- Reported results honestly, as a range across multiple tests, not a single best-case number.

## Tech stack

Python · DuckDB · pandas · scikit-learn · LightGBM · Hugging Face datasets

## Project structure

```
work/notebooks/          — weekly build notebooks (data contract, baseline, model, validation, action playbook, capstone)
work/capstone_report.md  — full written report
docs/index.html          — the deployed research paper
```

## How to reproduce

See `work/capstone_report.md`, Section 8, for exact setup steps and commands.

## About the Author

**Muhammad Muzammil** *Undergraduate Software & AI Engineering Student | Applied Machine Learning Engineer*

## Connect with Me

* **LinkedIn:** [linkedin.com/in/muzammilsharf](https://www.linkedin.com/in/muzammilsharf/)
* **GitHub:** [github.com/muzammilsharf](https://github.com/muzammilsharf)
* **Email:** [sharfmuzamil@gmail.com](mailto:sharfmuzamil@gmail.com)


## Data credit

Built on the FlyRank ML Internship dataset.