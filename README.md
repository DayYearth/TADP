# TADP: Learning with few target labels for cross-domain intrusion detection

Research notebooks accompanying **Learning with few target labels for cross-domain intrusion detection under episode-wise temporal evaluation**.

**Authors:** Nguyễn Quang Trung | Lê Huy Hoàng | Trần Hoàng Hải  
**Affiliation:** School of Information and Communication Technology, Hanoi University of Science and Technology, Hanoi, Vietnam

## Research question

When an intrusion detector moves to a new network, is it more useful to adapt source models without target labels or to label a small target sample and train directly?

The study compares seven methods without true target labels against **M2**, a compact classifier trained directly on a random sample of target-labelled flows. It separately examines whether updating the decision threshold improves detection.

## Data and evaluation

The study uses four datasets represented by 77 common numeric CICFlowMeter features:

| Dataset | Sample attack prevalence |
| --- | ---: |
| CIC-IDS2017, corrected DistriNet version | 24.1% |
| CSE-CIC-IDS2018, corrected DistriNet version | 5.6% |
| CIC-ToN-IoT | 53.0% |
| CIC-UNSW-NB15 | 2.5% |

Sampling targets approximately one million flows per dataset. The saved final-run log reports 999,712; 999,829; 1,000,028; and 1,000,399 flows, respectively.

A leave-one-domain-out experiment rotates the target over all four datasets. An episode is defined by UTC day and traffic-type label. Records are ordered by flow start time within each episode.

- Source-role partitions: 70% training, 10% validation, 20% test.
- Target-role partitions: earliest 10% adaptation, remaining 90% evaluation.
- M2 label budgets: 100, 500, 1,000, and 5,000, with three label draws per budget.
- Final experiment seeds: 11, 22, 33, 44, and 55.
- Classifier families: LightGBM, multilayer perceptron, and decision tree.
- Preprocessing: signed logarithm followed by standardisation fitted on the relevant training partition.
- Principal measures: Macro-F1, AUROC, false-positive rate, and detection rate.

The episode split preserves temporal order locally. It does not establish a single globally chronological deployment evaluation, and episode construction uses traffic-type annotations.

## Methods

The seven label-free alternatives are uniform source averaging, source-validation selection, TADP weighting, CORAL, Isolation Forest, self-training, and consensus pseudo-labelling.

M2 learns directly from target features and labels. Its initial threshold is selected using three-fold stratified out-of-fold predictions. Label draws with fewer than three records of either class return an all-benign predictor and remain in the Macro-F1 averages.

Threshold evaluation compares a fixed threshold, separation-gated Otsu (SG-Otsu), and false-positive-guarded Otsu (FPG-Otsu) at reference-sample limits of 5%, 10%, and 20%. The primary M2 rule is FPG-Otsu-10. It uses windows of 20,000 flows, a history of 50,000 scores, a 256-bin logit histogram, and a separation cutoff of 0.75.

In the notebooks, `a_co_dinh` denotes the fixed rule, `e_otsu_cua_so_co_chan` denotes SG-Otsu, and `h_chong_sap_05/10/20` denote the guard variants when a reference is available. For label-free methods without a benign reference, the adaptive rule only applies the separation gate. Pseudo-label methods use a pseudo-benign reference, which cannot certify true target FPR.

## Main reported results

At 500 target labels, M2 achieves:

| Model | Fixed Macro-F1 | FPG-Otsu-10 Macro-F1 |
| --- | --- | --- |
| LightGBM | 0.790 ± 0.020 | 0.826 ± 0.048 |
| MLP | 0.899 ± 0.007 | 0.895 ± 0.004 |
| Decision tree | 0.846 ± 0.031 | 0.819 ± 0.039 |

Values are means ± 95% confidence-interval half-widths over five seed means, after averaging label draws and equally weighting target domains.

The strongest label-free mean under the principal threshold rules is 0.558 ± 0.016. Across all model-budget groups, M2 wins 239 of 240 paired seed-domain comparisons against the family-specific best label-free adaptive comparator.

Threshold adaptation has model-dependent effects. It helps LightGBM on CIC-ToN-IoT, while reducing performance for several MLP and decision-tree configurations. The pooled LightGBM threshold comparisons do not establish a consistent improvement across all targets.

These values are reported experiment results preserved in notebook outputs. Publication of these notebooks does not constitute an independent rerun of the experiment.

## Notebook guide

| Notebook | Purpose |
| --- | --- |
| [05_GiaiDoan1_v3_CIC.ipynb](05_GiaiDoan1_v3_CIC.ipynb) | Dataset download, cache creation, and initial episode-split checks |
| [06_ChanDoan_UNSW.ipynb](06_ChanDoan_UNSW.ipynb) | UNSW threshold, labels, and feature diagnostics |
| [07_GiaiDoan1_v4_NguongNguon.ipynb](07_GiaiDoan1_v4_NguongNguon.ipynb) | Source-validation threshold experiments |
| [08_DoLech_DacTrung.ipynb](08_DoLech_DacTrung.ipynb) | Feature-shift and feature-removal diagnostics |
| [09_ItNhan.ipynb](09_ItNhan.ipynb) | Preliminary M1 source selection, M2 direct training, and M3 stacking |
| [09_ItNhan_v2_NguongKhopTyLe.ipynb](09_ItNhan_v2_NguongKhopTyLe.ipynb) | Preliminary prevalence-matching thresholds |
| [09_ItNhan_v3_BonCachNguong.ipynb](09_ItNhan_v3_BonCachNguong.ipynb) | Preliminary comparison of four threshold rules |
| [10_ChanDoan_CachChia.ipynb](10_ChanDoan_CachChia.ipynb) | Adaptation/evaluation distribution diagnostics |
| [11_KiemTra_ThuTu.ipynb](11_KiemTra_ThuTu.ipynb) | Timestamp ties and record-order diagnostics |
| [12_NguongCuaSo.ipynb](12_NguongCuaSo.ipynb) | Initial windowed threshold experiments |
| [12_NguongCuaSo_v2_ChanKep.ipynb](12_NguongCuaSo_v2_ChanKep.ipynb) | Earlier double-guard experiment |
| [12_NguongCuaSo_v3_ChongSap.ipynb](12_NguongCuaSo_v3_ChongSap.ipynb) | Guarded windowed Otsu and sensitivity experiments |
| [13_KhongNhan.ipynb](13_KhongNhan.ipynb) | Label-free method development |
| [14_BaMoHinh.ipynb](14_BaMoHinh.ipynb) | Three-model development experiment |
| [15_ChayChot.ipynb](15_ChayChot.ipynb) | Final five-seed experiment and summary statistics |

Notebook 15 is the entry point for the final paper results. Earlier notebooks document development and diagnostics; they are not a mandatory sequential execution pipeline.

## Interpretation and limitations

- Findings apply to these four datasets and the evaluated methods; advanced representation-learning and active-learning methods were not reimplemented.
- FPG-Otsu-10 limits an empirical reference-sample FPR. It does not guarantee a 10% deployment FPR.
- Seed confidence intervals quantify randomisation effects conditional on the datasets.
- Repeated seeds and draws share records and do not represent independent network deployments.
- The many-label diagnostic reference overlaps part of the evaluation partition and is excluded from the principal comparison.
- Some saved p-values are rounded. Holm-adjusted domain-level values in the manuscript are approximate where computed from rounded source p-values.
- Runtime, memory use, and operational alert-handling costs have not been benchmarked.

## Citation

Please cite the manuscript by its title and authors above. Publication venue, DOI, and final bibliographic details should be added when available.

