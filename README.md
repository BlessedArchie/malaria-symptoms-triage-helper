# Malaria Symptoms Triage Helper

## Project Overview
This project focuses on building a Predictive Classification Model to determine the urgency level (Low, Medium, High) a rural patient should assign to their symptoms before deciding whether to seek care, based on reported symptoms and basic demographic information.

Understanding when to seek care is critical in resource-limited rural settings. By identifying patients who need urgent attention, this tool can:
- Help rural patients make faster, more informed decisions about seeking care
- Reduce delays in treatment for severe cases
- Support (not replace) frontline health workers in prioritising who needs attention first

I use a Random Forest Classifier, benchmarked against a Logistic Regression baseline, to classify urgency level while keeping the model interpretable enough to explain its reasoning to non-clinical users.

**Important note:** This tool is a triage-support prototype only. It does not diagnose malaria or any other illness — it suggests when a patient may want to seek care based on reported symptoms.

## Dataset
The dataset is loaded directly from Google Drive into a Pandas DataFrame. It contains 30,000 patient records with 9 attributes capturing reported malaria symptoms (Fever, Headache, Chills, Fatigue), patient demographics (Age, Gender, Weight, Location), and the resulting severity outcome used to derive the urgency label.

| # | Feature | Type | Description |
|---|---|---|---|
| 1 | `Fever` | int64 | Whether the patient reported fever (0/1) |
| 2 | `Headache` | int64 | Whether the patient reported headache (0/1) |
| 3 | `Chills` | int64 | Whether the patient reported chills (0/1) |
| 4 | `Fatigue` | int64 | Whether the patient reported fatigue (0/1) |
| 5 | `Gender` | object | Gender of the patient |
| 6 | `Age` | int64 | Age of the patient |
| 7 | `Weight` | int64 | Weight of the patient (kg) |
| 8 | `Location` | object | Patient location (Rural/Urban) |
| 9 | `Severity` | object | **Target variable** — Severity outcome (Mild/Moderate/Severe), mapped to urgency level (Low/Medium/High) |

**Missing Values:** None — the dataset contains no missing values across any column.

**Important data note:** During exploration, we found that Severity in this dataset is fully determined by the number of reported symptoms (0–1 symptoms → Mild/Low, 2–3 → Moderate/Medium, 4 → Severe/High). This means the dataset behaves more like a rule-based synthetic dataset than real-world clinical data, where symptom severity is rarely this clear-cut. This is disclosed here rather than hidden, since it directly affects how the results below should be interpreted.

## Approach
1. **Exploratory Data Analysis (EDA):** examined class distribution, symptom patterns, and demographic spread; identified the deterministic symptom–severity relationship above.
2. **Encoding:** Severity mapped to an ordinal Urgency label (Low=0, Medium=1, High=2); Location and Gender binary-encoded.
3. **Feature Selection:** Mutual Information and Random Forest importance both confirmed that the four symptom features (Fever, Headache, Chills, Fatigue) carry nearly all predictive signal, while Age, Weight, Gender, and Location contribute almost nothing in this dataset.
4. **Modelling:** Two Random Forest classifiers were trained and compared — Model A (all 8 features) and Model B (the 4 selected symptom features) — using `class_weight='balanced'` to account for the underrepresentation of High-urgency cases (~6.4% of the dataset).
5. **Evaluation:** classification report (precision/recall/F1 per class), confusion matrix, and weighted multi-class AUC-ROC.
6. **Model Selection:** Model B (4 features) was selected as the final model.
7. **Stress Test:** hand-crafted symptom combinations were tested to sanity-check model behaviour beyond the training data's clean pattern.

## Results

| Metric | Model A (All Features) | Model B (Optimised — 4 Features) |
|---|---|---|
| Accuracy | 1.0 (100%) | 1.0 (100%) |
| Precision (weighted avg) | 1.0 (100%) | 1.0 (100%) |
| Recall (weighted avg) | 1.0 (100%) | 1.0 (100%) |
| F1-Score (weighted avg) | 1.0 (100%) | 1.0 (100%) |

**Confusion Matrix — Model B (Optimised):**

| | Predicted: Low | Predicted: Medium | Predicted: High |
|---|---|---|---|
| **Actual: Low** | 1,830 | 0 | 0 |
| **Actual: Medium** | 0 | 3,787 | 0 |
| **Actual: High** | 0 | 0 | 383 |

Both models achieved identical, perfect scores. Model B was selected as the final model because it achieves the same performance using only 4 inputs instead of 8 — reducing the information a patient or health worker needs to provide, which matters for quick use in rural, low-infrastructure settings.

## Limitations
- **Synthetic, deterministic labels:** Severity is fully determined by symptom count in this dataset, so the perfect evaluation scores reflect the model correctly learning that rule — not an ability to handle the noisier, less clear-cut symptom presentations of real-world malaria cases.
- **Symptom count over symptom combination:** stress testing showed the model responds to *how many* symptoms are reported rather than *which* symptoms are present — real clinical significance often depends on specific combinations, which this dataset does not capture.
- **Demographics have no effect on the current label:** Age and Location, despite being central to the rural-context framing of this project, do not currently influence urgency in the raw data. A production version would need this relationship built in deliberately or sourced from real clinical data.
- **No real-world clinical validation:** this model has not been tested against real patient outcomes and should not be used as an actual clinical decision-making tool.

## Disclaimer
This tool is a triage-support prototype and does not diagnose malaria or any other illness. It is intended to help suggest when a patient should seek medical care based on reported symptoms. It should never replace consultation with a qualified health worker, especially in cases of severe or worsening symptoms.

> ⚠️ This tool does NOT diagnose malaria or any medical condition. It only provides a general suggestion on how urgently you may need to seek care, based on the symptoms you report. It is not a substitute for professional medical advice. If symptoms are severe, worsening, or you are unsure, seek care from a qualified health worker immediately.





## Tools Used
Python, Pandas, scikit-learn, Google Colab, Matplotlib, Seaborn
