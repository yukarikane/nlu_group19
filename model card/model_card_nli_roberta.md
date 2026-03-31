---

## {}

language: en
license: cc-by-4.0
pipeline_tag: text-classification
tags:

- text-classification
- natural-language-inference
repo: 

---

# Model Card for COMP34812-NLI-RoBERTa

Binary NLI model: **RoBERTa** sequence classifier fine-tuned from an **MNLI-pretrained** checkpoint (`prajjwal1/roberta-base-mnli`) with a reinitialised **two-class** head.

## Model Details

### Model Description

The encoder weights start from `prajjwal1/roberta-base-mnli` (three-class NLI on MNLI); the classification head is replaced for binary labels with `ignore_mismatched_sizes=True`. Tokenization uses `roberta-base` with max length 256. Training uses the Hugging Face `Trainer` on course `train.csv` with dev-based checkpoint selection via macro-F1. Text passes through `clean_for_roberta` for stable tokenization. See `optimisation/roberta_improvement.md` for differences vs the course RoBERTa baseline.

- **Developed by:** Yihan Sun, Jianing Ren, Xiayuan Jin
- **Language(s):** English
- **Model type:** Supervised fine-tuning
- **Model architecture:** `RobertaForSequenceClassification` (RoBERTa-base, 12-layer encoder; binary classification head).
- **Finetuned from model [optional]:** Encoder initialised from `prajjwal1/roberta-base-mnli`; tokenizer `roberta-base`.

### Model Resources

- **Repository:** [https://huggingface.co/prajjwal1/roberta-base-mnli](https://huggingface.co/prajjwal1/roberta-base-mnli)
- **Paper or documentation:** RoBERTa: [https://arxiv.org/abs/1907.11692](https://arxiv.org/abs/1907.11692) — MNLI multi-genre NLI: [https://aclanthology.org/N18-1101/](https://aclanthology.org/N18-1101/)

## Training Details

### Training Data

COMP34812 NLI `train.csv` only (closed task). Rows cleaned with `clean_for_roberta` before tokenization.

### Training Procedure

#### Training Hyperparameters

- **Base checkpoint:** `prajjwal1/roberta-base-mnli` with `RobertaForSequenceClassification`, `num_labels=2`, `ignore_mismatched_sizes=True` (3-class head reinitialised to binary).
- **Tokenizer:** `roberta-base` (same notebook), `max_length=256`.
- **TrainingArguments:** `learning_rate=2e-5`, `per_device_train_batch_size=16`, `per_device_eval_batch_size=32`, `num_train_epochs=5`, `weight_decay=0.01`, `fp16=True`, `eval_strategy`/`save_strategy`=`epoch`, `metric_for_best_model=eval_macro_f1`, `load_best_model_at_end=True`, `logging_steps=100`.
- **Text:** `clean_for_roberta` (HTML unescape, NFKC, quote/dash normalisation, control-char stripping, whitespace) applied before tokenization.

#### Speeds, Sizes, Times

- **Training wall-clock (logged):** ~~845.5 s (~~14.09 min total) for 5 epochs on the recorded Colab run (`TrainOutput` in notebook).
- **Dev inference:** ~8.0 s for full dev set (`test_runtime` in `predict` metrics).
- **Saved weights:** `trainer.save_model` to `OUTPUT_DIR` (notebook default `/content/roberta_nli_output` on Colab); full RoBERTa checkpoints are typically a few hundred MB on disk.

## Evaluation

### Testing Data & Metrics

#### Testing Data

Course `dev.csv` (tokenized dev set; same labels as training).

#### Metrics

Accuracy and macro-F1 via `compute_metrics` (also logged each epoch during training).

### Results

Development set evaluation via `trainer.predict(dev_tokenized)` — metrics from `optimisation/Roberta_optimisation.ipynb`:

- **Accuracy:** 0.907957
- **Macro F1:** 0.907805

(Trainer reports these as `test_accuracy` / `test_macro_f1` on the dev batch.)

## Technical Specifications

### Hardware

GPU strongly recommended for fine-tuning (e.g. Colab T4-class); CPU feasible only for inference on small batches.

### Software

`transformers`, `torch`, `datasets`, `accelerate`, `scikit-learn` (versions per notebook `%pip install` / environment).

## Bias, Risks, and Limitations

MNLI-pretraining domain differs from the course dialogue-heavy data; binary head is randomly initialised. Possible tokenizer/version drift across environments. No demographic fairness analysis. Long inputs are truncated at 256 subwords.

## Additional Information

Evolved from `RoBEARTa baseline/Roberta_baseline.ipynb`; details in `optimisation/roberta_improvement.md`. Hugging Face model pages: [prajjwal1/roberta-base-mnli](https://huggingface.co/prajjwal1/roberta-base-mnli), [roberta-base](https://huggingface.co/roberta-base). Repository: [https://github.com/yukarikane/nlu_cwk](https://github.com/yukarikane/nlu_cwk).

**Fine-tuned weights (OneDrive — files >10 MB):** [https://livemanchesterac-my.sharepoint.com/:f:/g/personal/yihan_sun-2_student_manchester_ac_uk/IgA_RAm3gv9-S5aNuQeS6kRIAf4ttlI9UdRapvQsD9n5IVE?e=JjN4Yn](https://livemanchesterac-my.sharepoint.com/:f:/g/personal/yihan_sun-2_student_manchester_ac_uk/IgA_RAm3gv9-S5aNuQeS6kRIAf4ttlI9UdRapvQsD9n5IVE?e=JjN4Yn) (shared folder). `**Roberta_nli_output.zip`** — Hugging Face `Trainer` output for `OUTPUT_DIR` / `roberta_nli_output`: `checkpoint-`* folders (per-epoch saves with global step in the name, e.g. 1527, 4581, 7635), plus `config.json`, model weights (`pytorch_model.bin` and/or `model.safetensors`), tokenizer files, `training_args.bin`, `trainer_state.json`, and related Trainer metadata. Load the **best** checkpoint per `trainer_state.json` / `load_best_model_at_end` or use `save_model` output as in the notebook.