# Beyond a Single Benchmark: Phishing Website Detection Across Datasets

Code for the manuscript *"Beyond a Single Benchmark …"* (under double-blind review).
The study tests whether phishing detectors trained on one public dataset still work on others, shows a URL shortcut in PhiUSIIL, and trains a URL-only warning model that is checked on a newer outside test set.

> **Anonymous version.** Author names and links are removed for review and will be added after the review.

---

## Repository contents

```
.
├── notebooks/
│   ├── Revision_Experiments.ipynb   # E1–E5: all main paper and supplement tables/figures
│   └── URL_Guard.ipynb              # E6: Web Page only vs Web Page + PhiUSIIL, and the saved model
├── results/
│   ├── revision_results/            # outputs of Revision_Experiments.ipynb used in the paper
│   │   ├── repro/                   # config, versions, dataset hashes, retained rows, fold files
│   │   ├── tables/                  # fold-level CSVs, paired tests, LaTeX tables
│   │   ├── figures/                 # paper and supplement figures (PDF)
│   │   └── cleaned_data/            # cleaned PhiUSIIL and Web Page records with canonical URL and features
│   └── url_guard_results/           # outputs of URL_Guard.ipynb (tables, figures, repro, saved model)
├── LICENSE
└── README.md
```

`results/` holds the exact outputs used in the paper, so every number can be checked without running the code. Both notebooks run unchanged on **Kaggle**, **Google Colab** or a local PC; the platform is detected automatically.

## Experiments

| ID | What it checks | Notebook (section) | Paper |
|----|----------------|--------------------|-------|
| E1 | Dataset audit: duplicates, label conflicts, simple rules that separate the classes, domain overlap in random splits | Revision_Experiments (4, 5) | Sec. 4.1, Table 1, Fig. 1 |
| E2 | Random split vs. domain-grouped split, same features | Revision_Experiments (7b) | Sec. 4.2, Table 2 |
| E3 | Five models on four datasets, paired tests | Revision_Experiments (7, 12) | Sec. 4.3, Table 3, Fig. 2 |
| E4 | PhiUSIIL shortcut: feature-removal ladder, URL-shape rules, login/deep-link tests | Revision_Experiments (6b, 8) | Sec. 4.4, Fig. 3 |
| E5 | URL-only models (lexical features vs. character n-grams) tested on unseen sources and an outside set | Revision_Experiments (8b, 9) | Sec. 4.5, Table 4a |
| E6 | Training on Web Page alone vs. Web Page + PhiUSIIL, same settings | URL_Guard (4–9) | Sec. 4.6, Table 4b |

Learning curves (section 10) and prediction time (section 11) are in the supplement and Table 4c.

Main findings:

- In PhiUSIIL every legitimate URL is an `https://www.` homepage. Two one-line rules reach 99.68% accuracy; after this cue is removed a simple rule falls to about 70%.
- Random splits put the same domain in train and test and give higher scores than domain-grouped splits.
- Adding PhiUSIIL to Web Page raised MCC on the outside set (D4V) for all 12 model–input pairs, but also raised false alarms on legitimate deep links for all 12.

## Datasets

The raw data are **not** included (only the cleaned PhiUSIIL and Web Page records in `results/revision_results/cleaned_data/`). Download them from the sources below. The notebooks search all sub-folders and accept the file names listed, so files do not need to be renamed.

| Key | Dataset | Source | Accepted file names | Records (raw → used) |
|-----|---------|--------|---------------------|----------------------|
| PhiUSIIL | PhiUSIIL Phishing URL | UCI Machine Learning Repository | `PhiUSIIL_Phishing_URL_Dataset.csv` | 235,795 → 231,753 |
| WebPage | Web Page Phishing Detection | Mendeley Data, doi:10.17632/c2gw7fy2j4.3 | `Web_Page_Phishing_Detection.csv`, `dataset_phishing.csv`, `dataset_B_05_2020.csv` | 11,430 → 11,256 |
| PhishWeb | Phishing Websites | Mendeley Data (Vrbančič et al.) | `Phishing_Websites_Dataset.csv`, `Phishing_Websites_Dataset_1.csv`, `dataset_full.csv` | 88,647 → 87,209 |
| Tan | Phishing Dataset for Machine Learning | Mendeley Data, doi:10.17632/h3cgnj8hft.1 | `Phishing_Legitimate_full.csv` | 10,000 → 9,581 |
| Ext1000 (D4V) | PhishBench-External, outside test only | Mendeley Data, doi:10.17632/2ctjcnm9kn.1 | `Phishing_Detection_External_Evaluation_Dataset_1000_URLs.csv` | 642 URLs of unseen domains |

D4V is optional in `Revision_Experiments.ipynb` (skipped if missing) and required in `URL_Guard.ipynb`.

Cleaning: URLs are put in a standard form (NFKC; scheme, leading `www.` and trailing `/` removed), records with the same URL but different labels are dropped, and duplicates are removed. The SHA-256 of each input file and the kept row IDs are written to `repro/`.

## How to run

### Kaggle (used for the paper)

1. **+ Add Input** → attach the CSV files as a dataset.
2. Settings: **Internet ON** (for `pip install tldextract`), Accelerator: None (CPU is enough).
3. **Save Version → Save & Run All (Commit)**. The run continues if the browser is closed.
4. Download the zip from the **Output** tab: `revision_results/revision_outputs.zip` or `url_guard_results.zip`.

Run time on a Kaggle CPU: about 4–5 h for `Revision_Experiments.ipynb`. Both notebooks save each finished fold in `ckpt/`; if a run stops, attach the old output folder and run again, and finished folds are skipped.

### Google Colab

Put the CSV files anywhere in Google Drive and run all cells. Drive is mounted automatically, and results go to `MyDrive/revision_results` or `MyDrive/url_guard_results`.

### Local

```bash
pip install numpy pandas scipy scikit-learn xgboost lightgbm tldextract matplotlib joblib jupyter
DATA_DIR=/path/to/csv jupyter nbconvert --to notebook --execute notebooks/Revision_Experiments.ipynb
DATA_DIR=/path/to/csv jupyter nbconvert --to notebook --execute notebooks/URL_Guard.ipynb
```

`OUT_DIR` sets the output folder. `QUICK=1` runs a short test on a small sample.

## Settings

All settings were fixed before any test fold was seen and are saved in `repro/config.json`.

| Item | Value |
|------|-------|
| Cross-validation | 5 folds × seeds 42 and 44 (10 paired folds), grouped by registrable domain (tldextract, offline suffix list) |
| Random Forest | 200 trees, Gini, `max_features=sqrt` |
| SVM | StandardScaler + RBF, C=1, `gamma=scale`, at most 20,000 training rows |
| Decision Tree | Gini, no depth limit |
| XGBoost | 300 trees, library defaults |
| LightGBM | 300 trees, library defaults |
| Logistic Regression | C=1 (URL-only experiments) |
| Lexical URL features | 34 features from the URL string |
| Character n-grams | 3–5 chars, TF-IDF, `min_df=3`, up to 100k terms; top 2,000 by chi² for non-LR models; RF/DT on at most 60,000 rows |
| E6 source weights | Web Page and PhiUSIIL get the same total weight |
| Statistics | Nadeau–Bengio corrected t-test with Holm correction, McNemar test, domain-cluster bootstrap (1,000 resamples); Friedman/Nemenyi as descriptive only |
| Metrics | Accuracy, Precision, Recall, F1, F2, ROC-AUC, PR-AUC, MCC, balanced accuracy, FPR, FNR, training and prediction time |

## Outputs

**`Revision_Experiments.ipynb` → `revision_results/`**

| Folder / file | Contents |
|---------------|----------|
| `tables/all_fold_results.csv` | Every fold, model and dataset score |
| `tables/summary_mean_sd.csv` | Mean ± SD |
| `tables/paired_tests.csv` | Corrected t-tests, McNemar, Holm p-values |
| `tables/audit_*.csv`, `overlap_random_split.csv` | Dataset audit and split overlap |
| `tables/tab_*.tex` | LaTeX tables used in the paper and supplement |
| `figures/fig_*.pdf` | Figures used in the paper and supplement |
| `repro/` | `config.json`, `datasets.json` (SHA-256, counts), `versions.json`, fold files, retained rows |

**`URL_Guard.ipynb` → `url_guard_results/`**

| Folder / file | Contents |
|---------------|----------|
| `tables/all_fold_results.csv`, `summary_mean_std.csv` | All scores for both setups |
| `tables/paired_combined_vs_webpage.csv` | Combined − Web Page differences with Holm p-values |
| `tables/tab_combined.tex`, `deploy_candidates.csv` | Supplement table and model choice |
| `figures/fig_combined_auc.pdf`, `fig_combined_deeplink_fpr.pdf` | Supplement figures |
| `model/url_guard_model.joblib`, `model_card.json` | Saved model and its card |
| `repro/` | Config, combined URL list, fold files |

## Saved model (URL Guard)

`URL_Guard.ipynb` picks the model with the best mean ROC-AUC over the three test sets and retrains it on all 242,616 URLs. In our run this was **LightGBM on the 34 lexical features, trained on Web Page + PhiUSIIL** (mean ROC-AUC 0.901).

To score a URL, first run cell 3 of the notebook (`canonicalize`, `url_features`), then:

```python
import joblib, pandas as pd

b = joblib.load("results/url_guard_results/model/url_guard_model.joblib")

def score(url):
    c = canonicalize(url)
    if b["view"] == "CharNgram":
        X = b["vectorizer"].transform([c])
        if b["model_name"] != "LR":
            X = b["selector"].transform(X)
    else:
        X = pd.DataFrame([url_features(c)])[b["lex_features"]].values.astype("float32")
    p = b["model"].predict_proba(X)[0, 1]
    level = "low" if p < b["bands"]["low"] else ("high" if p >= b["bands"]["high"] else "medium")
    return p, level

print(score("http://paypa1-login.example-secure.xyz/verify"))
```

**Limits.** The model reads the URL text only. It flags about 23.5% of legitimate deep links (5.6–23.5% across models in our tests), mostly login and account pages. Use it to show a warning, not to block sites on its own.

## Software and hardware

Python 3.13, NumPy 2.1.3, pandas 2.3.3, SciPy 1.16.3, scikit-learn 1.6.1, XGBoost 3.4.1, LightGBM 4.6.0, tldextract 5.4.0, Matplotlib 3.10.0 (exact list in `repro/versions.json`).
All runs used the CPU of a Kaggle session. A GPU was attached to the session but not used.

## License

Code: MIT License. The datasets keep their own licenses; see each source.

## Citation

```bibtex
@article{anonymous2026beyond,
  title   = {Beyond a Single Benchmark: ...},
  author  = {Anonymous},
  journal = {Mesopotamian Journal of Cybersecurity},
  year    = {2026},
  note    = {Under review}
}
```
