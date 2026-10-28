# CGAP for Real-Time Physiological-State Recognition

## Project purpose

This project develops **CGAP (Context-Guided Adaptive Personalization)** for physiological-state recognition in future adaptive human–computer interaction systems. The system classifies wearable-sensor windows as baseline, stress, or amusement. Its main purpose is to combine general cross-participant learning with supervised online personalization for a new user.

The current implementation predicts physiological state. It does not directly control an interface, measure user satisfaction, or validate HCI task performance.

## Dataset

The project uses the WESAD wearable stress and affect detection dataset. The corrected working data contain 15 participants, 3,281 windows, 24 numerical features, and three classes: baseline, stress, and amusement.

The modalities are:

- ACC: body movement;
- BVP: cardiovascular response;
- EDA: sympathetic arousal;
- temperature: peripheral thermal response.

Each window is represented by mean, standard deviation, minimum, and maximum values. The 24 feature columns are:

```text
acc_1_mean, acc_1_std, acc_1_min, acc_1_max
acc_2_mean, acc_2_std, acc_2_min, acc_2_max
acc_3_mean, acc_3_std, acc_3_min, acc_3_max
bvp_mean, bvp_std, bvp_min, bvp_max
eda_mean, eda_std, eda_min, eda_max
temp_mean, temp_std, temp_min, temp_max
```

The metadata columns `subject`, `window`, `label`, and `label_id` are excluded from model training.

## Data preparation

The corrected participant-level CSV files are selected individually. The older combined CSV containing missing values is excluded. Before modelling, the pipeline checks required columns, labels, missing values, non-finite values, duplicates, participant counts, class frequencies, feature distributions, and correlations.

If missing values occur, median imputation is calculated from training participants only. Scaling is also fitted only on training participants. Extra-Trees does not require scaling; Logistic Regression, RBF SVM, and online SGD use training-derived scaling.

## Leakage-controlled evaluation

The evaluation is participant-independent. All windows from one participant remain in the same test partition. Each participant is held out once, while the other 14 participants are used for training and inner tuning. Three seeds are used: 17, 42, and 2026, producing 45 participant-seed runs. The participant, not the seed, is the independent statistical unit.

The online branch follows predict-then-update evaluation:

```text
predict current window → store prediction → reveal label → update model
```

Therefore, the current label is not used before its prediction. This is supervised online personalization, not completely label-free adaptation.

## Baseline models

- **Majority Dummy:** predicts the most frequent training class and provides a minimum reference.
- **Logistic Regression:** linear batch-learning reference.
- **RBF SVM:** nonlinear static batch-learning reference.
- **Global-only Extra-Trees:** measures cross-participant learning without personalization.
- **Online-only SGD:** measures sequential adaptation without probability fusion.

## Proposed CGAP architecture

```text
WESAD signals
    ↓
Window construction and 24 statistical features
    ↓
Training-only preprocessing and participant split
    ↓
 ┌─────────────────────────┐   ┌─────────────────────────┐
 │ Global branch           │   │ Online branch           │
 │ Extra-Trees             │   │ Scaled SGD              │
 │ Cross-participant       │   │ Predict, reveal, update │
 │ probabilities           │   │ User personalization   │
 └────────────┬────────────┘   └────────────┬────────────┘
              └──────────────┬─────────────┘
                             ↓
                    Probability fusion
                             ↓
              baseline / stress / amusement
                             ↓
                    Future HCI adaptation
```

The global branch learns general nonlinear relationships from the 24 features. The online SGD branch is initialized using training data and updates sequentially after labelled feedback. Their probability vectors are fused as:

```text
fused probability = (1 − λ) × global probability + λ × online probability
```

The class with the largest fused probability becomes the final prediction.

## Evaluation metrics

Accuracy reports the overall proportion of correct windows. Balanced accuracy averages recall across classes. Macro-precision, macro-recall, and macro-F1 give equal importance to baseline, stress, and amusement. Macro-F1 is the primary metric because accuracy can hide poor minority-class recognition.

Additional analyses include confusion matrices, ROC-AUC, calibration, reliability diagrams, online learning curves, sparse-feedback evaluation, ablation analysis, and SHAP explanations.

## Main results

| Model | Accuracy | Balanced accuracy | Macro-F1 | Macro-precision | Macro-recall |
|---|---:|---:|---:|---:|---:|
| Majority Dummy | 0.533 ± 0.007 | 0.333 ± 0.000 | 0.232 ± 0.002 | 0.178 ± 0.002 | 0.333 ± 0.000 |
| Logistic Regression | 0.533 ± 0.156 | 0.525 ± 0.156 | 0.477 ± 0.158 | 0.570 ± 0.142 | 0.525 ± 0.156 |
| RBF SVM | 0.587 ± 0.173 | 0.549 ± 0.146 | 0.508 ± 0.161 | 0.594 ± 0.184 | 0.549 ± 0.146 |
| **Proposed CGAP Full** | **0.946 ± 0.016** | **0.930 ± 0.016** | **0.936 ± 0.016** | **0.950 ± 0.012** | **0.930 ± 0.016** |

## Ablation results

| Configuration | Accuracy |  | Macro-F1 |
|---|---:|---:|---:|
| Global only | 0.629 |  | 0.481 |
| Proposed CGAP without online updates | 0.619 |  | 0.488 |
| Proposed CGAP without class weights | 0.929 |  | 0.909 |
| Online only | 0.946 |  | 0.936 |
| **Proposed CGAP Full** | **0.946** |  | **0.936** |

Removing online updates reduces macro-F1 from 0.936 to 0.488. The Online-only branch achieves the same rounded macro-F1 as CGAP Full, showing that online adaptation provides most of the predictive benefit in the current experiment.

## Sparse-feedback results

| Feedback condition | Macro-F1 |
|---|---:|
| Every window | 0.936 |
| Every 5 windows | 0.789 |
| Every 20 windows | 0.630 |
| No online updates | 0.488 |

Performance decreases when labelled feedback becomes less frequent. This is important for deployment because practical systems may not receive a label after every window.

## Per-class results

| Class | Precision | Recall | F1-score |
|---|---:|---:|---:|
| Baseline | 0.928 | 0.989 | 0.957 |
| Stress | 0.959 | 0.868 | 0.911 |
| Amusement | 0.963 | 0.919 | 0.940 |

Stress has the lowest recall, indicating overlap between stress and other physiological states.

## Explainability

Population-level SHAP plots aggregate explanations across all 15 participants. ACC and EDA statistics are among the strongest contributors, with BVP and temperature features also contributing depending on the class. SHAP values describe model attribution and are not evidence of physiological causation.

## Limitations and future work

1. WESAD contains controlled recordings from only 15 participants and does not directly measure HCI interaction, workload, satisfaction, or task performance.
2. Online learning uses labelled predict-then-update feedback rather than completely label-free adaptation.
3. The feature set uses statistical descriptors and does not include richer HRV, tonic–phasic EDA, frequency-domain, or learned temporal features.
4. External-dataset validation, long-term drift testing, energy measurement, and real HCI user studies remain future work.

## Reproducibility evidence

The project evidence consists of participant-level CSV files, EDA and quality-control diagrams, baseline results, CGAP LOSO outputs, ablation results, resumable checkpoints, calibration outputs, statistical tables, SHAP plots, architecture diagrams, and the Colab/Kaggle notebooks used for preprocessing and evaluation.

## Final interpretation

CGAP demonstrates a participant-independent framework that combines general physiological modelling with supervised online personalization. Its principal contribution is the complete leakage-controlled adaptive workflow, not the use of a new individual classifier. Further work must validate realistic label acquisition, temporal robustness, external generalization, computational efficiency, and direct HCI benefit.
