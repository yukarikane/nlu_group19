# COMP34812 NLU Coursework — NLI Solutions

**Group:** Yihan Sun, Jianing Ren, Xiayuan Jin (Group 19)  
**Repository:** [https://github.com/yukarikane/nlu_group19](https://github.com/yukarikane/nlu_group19)

This repository contains **two distinct solutions** for the **Natural Language Inference (NLI)** track (Track A): a **traditional ML** pipeline (TF-IDF + LinearSVC) and a **Transformer** fine-tuning pipeline (RoBERTa with MNLI pre-training). Both use only the **course-provided** training and development data (closed task).

---

## Repository layout

Paths below are relative to this folder (`nlu_group19/`), the group submission bundle.


| Path                                   | Description                                             |
| -------------------------------------- | ------------------------------------------------------- |
| `code/nlu_tf_idf_optimised.ipynb`      | Optimised TF-IDF + LinearSVC pipeline (category **A**)  |
| `code/Roberta_optimisation.ipynb`      | Optimised RoBERTa fine-tuning pipeline (category **C**) |
| `model card/model_card_nli_tfidf.md`   | Model card (Deliverable 3) for the TF-IDF solution      |
| `model card/model_card_nli_roberta.md` | Model card (Deliverable 3) for the RoBERTa solution     |
| `prediction/Group_19_A.csv`            | Test predictions for category **A** (Deliverable 1)     |
| `prediction/Group_19_C.csv`            | Test predictions for category **C** (Deliverable 1)     |
| `README.md`                            | This file (usage, data layout, deliverables)            |


Course CSVs (`train.csv`, `dev.csv`, `test.csv`), baseline notebooks, large checkpoints, and optional helpers are **not** shipped inside this folder; during development they usually live next to the main coursework repo (e.g. `training_data/NLI/` at the parent `nlu_cwk/` root) or paths you set in the notebooks. See **Data** and **How to run** below.

---

## Environment

- **Python**: 3.10+ recommended (3.11/3.12 work for the notebooks we tested).
- **TF-IDF solution**: `numpy`, `pandas`, `scipy`, `scikit-learn`, `joblib` (install cells are at the top of the notebook).
- **RoBERTa solution**: `transformers`, `datasets`, `evaluate`, `accelerate`, `torch` with a **GPU** strongly recommended (e.g. Google Colab T4).
- **Model cards (optional regeneration)**: `jinja2`, `huggingface_hub` (see `optimisation/generate_model_cards.py`).

---

## Data

1. **Source:** All training and evaluation data come from the **University of Manchester COMP34812** NLI coursework release (`train.csv`, `dev.csv`, and `test.csv` when provided). We did **not** download or merge any **external** NLI or general text datasets (closed-task requirement).
2. **Layout:** Place `train.csv` and `dev.csv` under `training_data/NLI/` (project root), **or** set the environment variable `NLU_DATA_DIR` to the folder that contains those files.
3. For **test inference**, either:
  - put `test.csv` in `test_data/NLI/test.csv`, or  
  - use the same data directory as training and add `test.csv` next to `train.csv` / `dev.csv`.

Column names expected: `premise`, `hypothesis`, and `label` for train/dev; test typically has `premise` and `hypothesis` only.

---

## How to run

### 1) TF-IDF + LinearSVC (`optimisation/nlu_tf_idf_optimised.ipynb`)

1. Open the notebook from the **project root** `nlu_cwk/` (so paths like `training_data/NLI/` resolve), or from `optimisation/` (paths fall back to `../training_data/NLI`).
2. Run cells in order:
  - **§1 Training**: fits vectoriser + classifier, then saves  
  `optimisation/nli_tfidf_artifacts/tfidf_linsvc.joblib`.
  - **§2 Evaluation**: metrics on the development set.
  - **§3 Test inference**: writes `tf_IDF_predictions.csv` if a test file is found.  
  If the joblib file exists, the test section can reload the model without re-running training.

### 2) RoBERTa (`optimisation/Roberta_optimisation.ipynb`)

Designed for **Google Colab** but runnable locally if paths and GPU are set up.

1. Install dependencies (first code cell).
2. **Google Drive** (optional): the `drive.mount` cell is skipped automatically when not on Colab.
3. **Data path**: the notebook tries: `/content/drive/MyDrive/nlu_cwk_data/train.csv` and `/content/drive/MyDrive/nlu_cwk_data/dev.csv`. This path uses MyDrive. Please create such a repository or change the data path to the one you need.
4. Run **§1 Training** → **§2 Evaluation (dev)** → **§3 Save + reload** → **test inference**.
  Checkpoints live under `OUTPUT_DIR` (default `/content/roberta_nli_output` on Colab); the reload cell resolves `MODEL_PATH` to the best checkpoint or the latest `checkpoint-`* if the root folder has no full save.

**Pre-trained weights (not trained by us from scratch):** the encoder is initialised from `[prajjwal1/roberta-base-mnli](https://huggingface.co/prajjwal1/roberta-base-mnli)` on the Hugging Face Hub; the tokenizer follows `[roberta-base](https://huggingface.co/roberta-base)`. See **Attribution** below.

Test output file: `roberta_predictions.csv`.

---

## Deliverables (course convention)

- **Predictions (Deliverable 1)**  
Submit CSVs via Canvas as `Group_<n>_<CATEGORY>.csv`, where `<CATEGORY>` is **A** (traditional ML) or **C** (Transformer) per the coursework spec.  
Ensure the CSV format matches the official instructions (typically a single `label` column).
- **Code (Deliverable 2)**  
These notebooks separate **training**, **evaluation on dev**, and **test inference** in labelled sections.  
Trained artefacts:
  - TF-IDF: `optimisation/nli_tfidf_artifacts/tfidf_linsvc.joblib`
  - RoBERTa: Hugging Face `save_model` / `checkpoint-`* under your `OUTPUT_DIR` (zip and upload if required).
- **Model cards (Deliverable 3)**  
Markdown model cards for each solution:
  - `optimisation/model_card_nli_tfidf.md`
  - `optimisation/model_card_nli_roberta.md`  
  They were produced with the course template `[Archive/COMP34812_modelcard_template.md](Archive/COMP34812_modelcard_template.md)` using `huggingface_hub.ModelCard.from_template` (see `[Archive/Model Card Creation.ipynb](Archive/Model%20Card%20Creation.ipynb)`). To regenerate after changing metrics in the notebooks, run from the repo root:
  ```bash
  pip install jinja2 huggingface_hub   # if needed
  python optimisation/generate_model_cards.py
  ```

---

## Trained models on the cloud (OneDrive / large files)

**Course rule:** Do **not** upload individual files **larger than 10MB** to Canvas. We host large trained artefacts on **University OneDrive / SharePoint** and link them here.

**Shared folder (TF-IDF + RoBERTa):**  
[https://livemanchesterac-my.sharepoint.com/:f:/g/personal/yihan_sun-2_student_manchester_ac_uk/IgA_RAm3gv9-S5aNuQeS6kRIAf4ttlI9UdRapvQsD9n5IVE?e=JjN4Yn](https://livemanchesterac-my.sharepoint.com/:f:/g/personal/yihan_sun-2_student_manchester_ac_uk/IgA_RAm3gv9-S5aNuQeS6kRIAf4ttlI9UdRapvQsD9n5IVE?e=JjN4Yn)


| Path / file in the share                   | Contents                                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tf_idf/tf-idf_model/nli_tfidf_artifacts/` | `tfidf_linsvc.joblib` — fitted `TfidfVectorizer` + `LinearSVC` (same relative layout as `optimisation/nli_tfidf_artifacts/` in the repo).                                                                                                                                                                                                                                                                |
| `Roberta_nli_output.zip`                   | Hugging Face `Trainer` output for fine-tuned RoBERTa: `checkpoint-`* directories (per-save steps, e.g. 1527, 4581, 7635), `config.json`, model weights (`pytorch_model.bin` / `model.safetensors`), tokenizer files, `training_args.bin`, `trainer_state.json`, etc. Unzip and set `MODEL_PATH` / load the **best** checkpoint per `trainer_state.json` (see `optimisation/Roberta_optimisation.ipynb`). |


If the link stops working after submission, use the same paths inside a refreshed share as required by the course team.

---

## Attribution

We explicitly acknowledge the following sources. **Reusing course or third-party code without attribution is academic malpractice**; this section is kept accurate to our submission.

### Data

- **COMP34812 NLI dataset** (training / development / test as released for this coursework): provided by the course team (University of Manchester). **No external datasets** were used for training or evaluation beyond this release (closed task).

### Course code and materials reused

- **TF-IDF track:** Our implementation **extends** the official baseline in `[IF-IDF baseline/nlu_tf_idf.ipynb](IF-IDF%20baseline/nlu_tf_idf.ipynb)` (separate premise/hypothesis TF-IDF, extra features, `LinearSVC` settings, and paths as documented in `optimisation/tf-idf_improvement.md`).
- **RoBERTa track:** Our implementation **extends** the official baseline in `[RoBEARTa baseline/Roberta_baseline.ipynb](RoBEARTa%20baseline/Roberta_baseline.ipynb)` (MNLI checkpoint choice, cleaning, `TrainingArguments`, and evaluation flow as in `optimisation/roberta_improvement.md`).
- **Model card template and example workflow:** `[Archive/COMP34812_modelcard_template.md](Archive/COMP34812_modelcard_template.md)` and `[Archive/Model Card Creation.ipynb](Archive/Model%20Card%20Creation.ipynb)` (course-provided).

### Pre-trained models and libraries (third party)

- **Hugging Face Hub:** Encoder weights loaded from `[prajjwal1/roberta-base-mnli](https://huggingface.co/prajjwal1/roberta-base-mnli)` (MNLI fine-tune); tokenizer from `[roberta-base](https://huggingface.co/roberta-base)`. RoBERTa is described in [Liu et al., 2019](https://arxiv.org/abs/1907.11692); MNLI in [Williams et al., 2018](https://aclanthology.org/N18-1101/).
- **Software:** [Hugging Face Transformers](https://huggingface.co/docs/transformers), [Datasets](https://huggingface.co/docs/datasets), [scikit-learn](https://scikit-learn.org/), [pandas](https://pandas.pydata.org/), [PyTorch](https://pytorch.org/), [NumPy](https://numpy.org/), [SciPy](https://scipy.org/), and other packages pinned or printed in our notebook install cells.

---

## Use of generative AI tools

We used **generative AI assistants** (e.g. **ChatGpt**) **sparingly** and only as a **supporting** tool. Typical uses were: polishing **README** and **model card** wording, suggesting **notebook section headings** (§1 Training / §2 Evaluation / §3 Demo) and **inline comments**, explaining **Transformers / Colab** error messages so we could fix them ourselves, and drafting a small `**generate_model_cards.py`** helper that wraps `huggingface_hub.ModelCard.from_template`—we **reviewed and ran** all code and kept changes that matched the course template and our notebooks.

We did **not** use generative AI to obtain **labels**, to access **unreleased test answers**, or to replace **running our own experiments**. Design choices (e.g. separate TF-IDF encodings and hand-crafted pair features for NLI; RoBERTa initialised from `roberta-base-mnli` with a binary head; dev-set metrics for checkpoint selection) and all **reported numbers** come from **our training and evaluation runs**. We understand these as standard **supervised** pipelines: sparse linear classification on engineered features for the traditional model, and **fine-tuning** a pretrained encoder for the Transformer model.

---

## Licence / course rules

Follow the latest **COMP34812** coursework brief regarding submission deadlines, group numbers, naming (`Group_19_A` / `Group_19_C`, etc.), and closed-task constraints.