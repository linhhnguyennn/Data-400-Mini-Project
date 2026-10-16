# Data-400-Mini-Project
# American Cinema in a Global Context

**Do American films get rated differently from films made elsewhere?** A Data 400 mini-project comparing Letterboxd average ratings of American and non-American films.

Author: Linh Nguyen · Course: Data 400 · Dickinson College · Oct 15 2026

## Overview

This project connects my Data Analytics major with American Studies and my Film Studies minor by treating film as cultural production. Using 87,286 rated films from a public Letterboxd dataset, it asks:

> How do Letterboxd average ratings of American films compare with those of films made outside the United States, and does the difference vary by genre?

It follows the Data 400 workflow: frame the problem, get the data, split before exploring, explore and visualize, prepare the data in a pipeline, compare models using cross-validation, tune the best model, evaluate once on a held-out test set, and report limitations honestly.

### Key findings

- US-involved films are rated about **0.16 stars lower** than non-US films (95% CI -0.165 to -0.154) after controlling for genre, release year, and runtime.
- The gap appears in all 19 genres. It is largest in Family and Horror and reverses in Documentary.
- The result holds under every way of classifying co-productions, and in Lasso, OLS, and Low/Medium/High rating-tier models.
- Cast size and a count of prolific actors do not explain the gap (the coefficient changes from -0.170 to -0.166).
- The best model (histogram gradient boosting) reaches a **test RMSE of 0.315** against 0.421 for a mean-rating baseline (R² = 0.44).
- This is an association that likely reflects who rates which films on Letterboxd, not evidence that American films are worse.

## Data source

**Letterboxd (Movies Dataset)** on Kaggle: <https://www.kaggle.com/datasets/gsimonx37/letterboxd>

The notebook downloads these files automatically, joined by a film `id`:

| File | Columns used | Purpose |
|---|---|---|
| `movies.csv` | `id`, `date`, `minute`, `rating` | One row per film. `rating` (average rating) is the target. |
| `countries.csv` | `id`, `country` | Defines origin: US only, US co-production, or non-US |
| `genres.csv` | `id`, `genre` | 19 genre indicators |
| `themes.csv` | `id`, `theme` | Top 20 themes (optional add-on) |
| `actors.csv` | `id`, `name` | Cast size and prolific-actor features (extension) |

Please check the dataset page for its license and terms of use. **The data files are not included in this repository** because they are large. The notebook downloads them each time it runs.

**Origin rule:** a film is *US only* if "USA" is its only listed country, a *US co-production* if "USA" appears with other countries, and *non-US* otherwise.

## Repository contents

```
.
├── README.md                      # this file
├── requirements.txt               # Python packages
├── DATA_400_Mini_Project.ipynb    # full analysis notebook (run top to bottom)
├── report/                        # written report (PDF or Word export)
├── slides/                        # presentation (optional)
└── figures/                       # exported plots (optional)
```

Rename the notebook file above to match your actual filename.

### What the notebook contains

| Section | What it does |
|---|---|
| 1. Get the data | Downloads the files, shows the first rows, checks missing values and country spellings |
| 2. Build one row per film | Cleans years, applies the origin rule, builds genre indicators |
| 3. Split | 80/20 train/test split stratified by origin and decade, with themes chosen from the training set only |
| 4. Exploratory analysis | Rating distributions, decade trends, genre gaps, genre and theme frequencies, theme and cast coverage (training set only) |
| 5. Models | One preprocessing pipeline; baseline, Ridge, Lasso, decision tree, random forest, and gradient boosting compared with 5-fold cross-validation |
| 6. Tuning | Randomized hyperparameter search for the best model |
| 7. Test evaluation | One-time evaluation on the held-out test set, with permutation importance |
| 8. US effect | OLS regression with controls, genre-by-origin interactions, and a co-production robustness check |
| 9. Rating tiers | Low/Medium/High classification with tercile cutoffs from the training set |
| 10. Casting | Does adding cast features change the US effect or improve prediction? |

## Required software

- **Python 3.10 or newer** (developed in Google Colab with Python 3.13)
- **Jupyter or Google Colab** (the notebook uses `display()`, which needs a notebook environment)
- **Python packages** (listed in `requirements.txt`): `pandas`, `numpy`, `scikit-learn`, `statsmodels`, `matplotlib`, `seaborn`, `kagglehub`
- **Internet access** to download the dataset from Kaggle

## How to run the analysis

### Option A: Google Colab (recommended)

1. Go to <https://colab.research.google.com>, choose **File → Open notebook → GitHub**, and open `DATA_400_Mini_Project.ipynb` from this repository. Or download the notebook and use **File → Upload notebook**.
2. Optional: choose **Runtime → Change runtime type → High-RAM** if the session runs out of memory.
3. Run the first cell to install and import packages (`kagglehub` is installed with `pip` in that cell).
4. Run the data-loading cell. It clears any half-downloaded cache and downloads the five files. If a download is corrupted, run the cell again.
5. Run the remaining cells **in order, from top to bottom**. Later cells depend on variables from earlier ones.
6. When finished, use **Runtime → Restart session and run all** to confirm the whole notebook runs cleanly.

### Option B: Local Jupyter

1. Clone the repository and move into it:
```bash
   git clone https://github.com/[your-username]/[your-repo].git
   cd [your-repo]
```
2. Create an environment and install the packages:
```bash
   python -m venv venv
   source venv/bin/activate      # on Windows: venv\Scripts\activate
   pip install -r requirements.txt
```
3. Start Jupyter and open the notebook:
```bash
   jupyter notebook DATA_400_Mini_Project.ipynb
```
4. In the first cell, remove the `!pip install` line if you installed packages already, and in the data cell remove the line that deletes `/root/.cache/kagglehub/...` (it is Colab-specific).
5. Run all cells in order.

### If the Kaggle download asks for credentials

Public datasets usually download without a login. If you are asked for credentials, create an API token in your Kaggle account (**Settings → API → Create New Token**) and provide it through Colab Secrets (`KAGGLE_USERNAME` and `KAGGLE_KEY`) or environment variables. **Never commit `kaggle.json` or any credentials to the repository.**

## Notes on reproducibility and integrity

- All random operations use `random_state=42`, so splits and models are reproducible.
- The train/test split happens **before** any exploration. Exploration, model comparison, and tuning use the training set only.
- Imputation, encoding, and scaling live inside one scikit-learn pipeline fit on training data only, to avoid leakage.
- The test set is used **once per task** (regression, tier classification, and the cast comparison). Do not change modeling choices after running a test-set cell.
- Engagement measures such as watches and likes are deliberately excluded because they are outcomes of a film's reception (target leakage).
- The slowest cells are the cross-validated model comparison (especially random forest) and the hyperparameter search. Reduce `n_estimators` or `n_iter` for a quicker test run.

## Limitations

- The dataset has no number-of-ratings variable, so some average ratings are noisy.
- Letterboxd users are a self-selected audience, so ratings reflect their tastes.
- Results show association, not cause.
- "American" is defined by listed production countries, not by a film's setting, story, or funding.
- All non-US countries are grouped together, which hides differences between national cinemas.
- Theme data is missing for most films and differs by origin (about 14% coverage for non-US films vs. 39% for US-only), so the main model uses genres only.
- Cast features are crude proxies, and rating-tier cutoffs are arbitrary.

See the written report in `report/` for the full discussion.

## Acknowledgments

Dataset: Letterboxd (Movies Dataset) on Kaggle, <https://www.kaggle.com/datasets/gsimonx37/letterboxd>. Analysis for Data 400 at [Institution]. Coursework in American Studies, Film Studies, and Data Analytics informed the framing.
