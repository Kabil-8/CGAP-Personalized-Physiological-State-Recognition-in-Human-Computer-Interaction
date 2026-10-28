CGAP Statistical Significance Analysis
Input CSV: /kaggle/input/datasets/kabil08/checkpoints/cgap_fast_ablation_checkpoints-20261027T050652Z-1-001/cgap_fast_ablation_checkpoints/results_by_seed.csv
Input rows: 15
Participants represented: 1
Seeds represented: [17, 42, 2026]
Models: ['CGAP Full', 'Global only', 'Online only', 'CGAP no online updates', 'CGAP no class weights']

Analysis unit: participant. Seeds averaged within participant before testing.
Friedman is the omnibus test; Wilcoxon tests compare CGAP Full with each comparator.
Holm adjustment is applied separately for each metric across its post-hoc comparisons.
Bootstrap confidence intervals are for paired participant-level mean differences.
Positive CGAP-minus-comparator differences favor CGAP when higher metric values are better.
Do not interpret missing/incomplete participant coverage as a completed 15-subject experiment.

Output files:
- descriptive_statistics.csv
- friedman_omnibus.csv
- paired_values_accuracy.csv
- paired_values_balanced_accuracy.csv
- paired_values_macro_f1.csv
- paired_values_macro_precision.csv
- paired_values_macro_recall.csv
- participant_means_across_seeds.csv
- sign_flip_permutation_supplementary.csv
- wilcoxon_posthoc_holm.csv