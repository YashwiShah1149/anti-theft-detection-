Anti-Theft Detection: Sensor-Based Model Evaluation

Does adding phone-status signals (SIM change, airplane mode, switch-off) to motion-sensor data improve theft detection — and reduce false alarms when a friend or family member just picks up the phone?

Problem
Most phone anti-theft systems rely on motion sensors alone (accelerometer / gyroscope). This breaks down in a common everyday scenario: a friend or family member picks up and handles the phone, producing motion that looks a lot like "theft-like" motion to a sensor-only model — a false positive.

Idea Tested
Add a small set of phone-status signals — SIM card change, airplane mode toggled, phone switched off, screen/lock state — alongside the motion sensors, and measure whether that improves theft vs. normal-motion classification.

Data
Collected with the Phyphox app: gyroscope + linear acceleration (and, for Model B, phone-status events), across:

9 participants
119 activities (walking, running, normal handling, theft-like grab/snatch motions, etc.)
174K+ raw sensor readings

Models Compared

Same participants and activities for both:
Model	Features
Model A	Sensor-only (gyroscope + linear acceleration)
Model B	Sensor + phone-status (SIM change, airplane mode, switch-off)

Method
Sliding-window feature extraction — raw sensor streams are chunked into fixed-size overlapping windows, each reduced to summary statistics (mean/std/min/max) plus HAR features (RMS, Signal Magnitude, SMA).
Classifiers — Random Forest is the headline model, benchmarked against a majority-class baseline, Logistic Regression, and XGBoost.
Leave-one-participant-out (LOPO) cross-validation — each fold trains on 8 participants and tests on the held-out participant, so results reflect generalization to an unseen person's motion patterns.
Paired significance testing — paired t-test and Wilcoxon signed-rank test, with Bonferroni correction across 4 metrics (α = 0.05 → 0.0125).
Bootstrap confidence intervals — 95% CI on the improvement from 10,000 resamples of the paired per-participant differences.
Headline Result

Adding phone-status features improved Random Forest performance:

Metric	Model A (sensor-only)	Model B (sensor + phone-status)
Accuracy	83.6%	88.3%
Precision	88.3%	90.2%
Recall	80.8%	88.0%
F1	84.4%	89.0%

With only 9 participants and a Bonferroni-corrected threshold, the paired significance tests did not provide enough evidence to call the improvement statistically significant, even though the average improvement was substantial — the notebook is upfront about this rather than only reporting the headline numbers.

Repository Structure
.
├── fianl_anti_theft_notebook.ipynb   # Main notebook: data loading, feature
│                                       engineering, model training, LOPO
│                                       evaluation, statistical testing
├── requirements.txt                   # Python dependencies
└── README.md

Getting Started

1. Install dependencies
pip install -r requirements.txt

2. Prepare your data
The notebook expects two folders of raw Phyphox exports (.xls/.xlsx), one per model:
Model A folder — one subfolder per participant, each containing recordings with a Gyroscope and Linear Acceleration sheet.
Model B folder — same structure, plus a Phone status sheet in each file.
Recording filenames should indicate the activity (e.g. contain "theft", "steal", "snatch" for theft-like motions, or "friend" for a friend/family member handling the phone, which is always treated as normal handling).
Update the two paths in the "Load the two datasets" section of the notebook to point at your local folders:
python
modelA_path = r"path/to/Model A data"
modelB_path = r"path/to/Model B data"

3. Run the notebook
jupyter notebook fianl_anti_theft_notebook.ipynb
Run all cells top to bottom. The notebook prints a reproducibility check (total window counts) that should be identical across runs on the same data — if not, files are being read inconsistently and results shouldn't be trusted until that's fixed.
