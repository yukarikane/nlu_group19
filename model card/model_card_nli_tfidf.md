---

## {}

language: en
license: cc-by-4.0
pipeline_tag: text-classification
tags:

- text-classification
- natural-language-inference
repo:[https://github.com/yukarikane/nlu_group19](https://github.com/yukarikane/nlu_group19)

---

# Model Card for COMP34812-NLI-TFIDF

Binary natural language inference (entailment vs not): traditional ML with dual TF-IDF encodings, directional sparse features, 21-dim linguistic pair features, and a **LinearSVC** classifier.

## Model Details

### Model Description

This model targets the COMP34812 closed-book NLI track. It extends the course TF-IDF baseline by encoding premise and hypothesis **separately**, adding **difference** and **Hadamard product** of the two TF-IDF vectors for directionality, and **21** hand-engineered features motivated by NLI (modals, quantifiers, negation, char n-gram overlap). The classifier is **LinearSVC** (`scikit-learn`). No external datasets beyond the provided `train.csv`.

- **Developed by:** Yihan Sun, Jianing Ren, Xiayuan Jin
- **Language(s):** English
- **Model type:** Supervised
- **Model architecture:** Sparse linear model: `TfidfVectorizer` + stacked sparse features + `LinearSVC` (not a neural encoder).
- **Finetuned from model [optional]:** None (linear classifier on engineered features; not fine-tuned from a pretrained transformer).

### Model Resources

- **Repository:** [https://github.com/yukarikane/nlu_cwk](https://github.com/yukarikane/nlu_cwk)
- **Paper or documentation:** [https://scikit-learn.org/stable/modules/generated/sklearn.svm.LinearSVC.html](https://scikit-learn.org/stable/modules/generated/sklearn.svm.LinearSVC.html)

## Training Details

### Training Data

COMP34812 NLI training split `train.csv` (~24 432 rows, English premise–hypothesis pairs with binary labels). No additional corpora. Preprocessing follows the optimisation notebook (pair features + separate TF-IDF as described in `optimisation/tf-idf_improvement.md`).

### Training Procedure

#### Training Hyperparameters

- **TfidfVectorizer:** `lowercase=True`, `stop_words=None`, `ngram_range=(1, 2)`, `min_df=2`, `max_df=0.95`, `sublinear_tf=True`; separate vectoriser fitted on premise/hypothesis corpus; features stacked with sparse **diff** (P−H), **element-wise product** (P⊙H), and **21-dim** hand-crafted pair features (overlap, negation, numbers, modals, quantifiers, double-negation heuristics, char 3-gram overlap).
- **LinearSVC:** `C=1.0`, `max_iter=5000`.
- **Feature dimensionality (train):** sparse stack dimension printed in notebook (order of 10^5 TF-IDF columns plus 21 dense).

#### Speeds, Sizes, Times

- **Training:** not timed in the notebook; fitting `LinearSVC` on the stacked sparse matrix is CPU-bound (typically minutes on a modern laptop).
- **Saved artefact:** `optimisation/nli_tfidf_artifacts/tfidf_linsvc.joblib` (created by the notebook after training). Size depends on fitted vocabulary and coefficients; measure locally with `ls -lh` after export.

## Evaluation

### Testing Data & Metrics

#### Testing Data

Official course development set `dev.csv` (6736 examples), same schema as training.

#### Metrics

Accuracy; macro-averaged F1; binary F1; per-class precision/recall/F1 from `sklearn.metrics.classification_report`.

### Results

Development set (`dev.csv`, 6736 examples) — values from `optimisation/nlu_tf_idf_optimised.ipynb` cell output:

- **Accuracy:** 0.645487
- **F1 (macro):** 0.644882
- **F1 (binary):** 0.659538

Per-class summary (from the same run) is printed in the notebook (`classification_report` for *not entailment* vs *entailment*).

## Technical Specifications

### Hardware

CPU sufficient; RAM scales with sparse matrix size during fitting (16 GB+ recommended for comfort).

### Software

Python 3.11; `numpy`, `pandas`, `scipy`, `scikit-learn`, `joblib` (versions as in `nlu_tf_idf_optimised.ipynb` install cell output, e.g. scikit-learn 1.7.x).

## Bias, Risks, and Limitations

Bag-of-words and TF-IDF miss deep semantics and long-range dependencies; lexical overlap can be misleading. Domain is informal English dialogue/text from the course data; no cross-lingual or fairness audit. Long sequences are represented via n-grams but without explicit length truncation beyond vectoriser behaviour.

## Additional Information

Course baseline extended from `IF-IDF baseline/nlu_tf_idf.ipynb`; design notes: `optimisation/tf-idf_improvement.md`. Project repository: [https://github.com/yukarikane/nlu_cwk](https://github.com/yukarikane/nlu_cwk).

**Trained artefact (OneDrive — files >10 MB):** [https://livemanchesterac-my.sharepoint.com/:f:/g/personal/yihan_sun-2_student_manchester_ac_uk/IgA_RAm3gv9-S5aNuQeS6kRIAf4ttlI9UdRapvQsD9n5IVE?e=JjN4Yn](https://livemanchesterac-my.sharepoint.com/:f:/g/personal/yihan_sun-2_student_manchester_ac_uk/IgA_RAm3gv9-S5aNuQeS6kRIAf4ttlI9UdRapvQsD9n5IVE?e=JjN4Yn) (shared folder). TF-IDF export path inside the share: `tf_idf/tf-idf_model/nli_tfidf_artifacts/` — contains `tfidf_linsvc.joblib` (fitted `TfidfVectorizer` + `LinearSVC`).