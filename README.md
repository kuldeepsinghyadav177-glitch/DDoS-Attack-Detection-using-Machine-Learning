# DDoS Attack Detection in Network Traffic using Machine Learning (CICIDS2017)

**Authors:** Kuldeep Yadav, Jai Dhurve, Bhuvan, Ayush Joshi

## 1. Abstract of the project

Distributed Denial-of-Service (DDoS) attacks flood a target with traffic and make services unavailable to legitimate users. In this project we build a supervised machine-learning pipeline that classifies network flows as either `BENIGN` or `DDoS`. We use the Friday-afternoon DDoS capture of the CICIDS2017 dataset (225,745 flows, 79 columns). After cleaning the data, removing data-leakage features and highly collinear features, we train a K-Nearest Neighbours (KNN) classifier and a Random Forest (RF) classifier. KNN reaches about 99.63% accuracy (167 errors on 44,617 test flows) and Random Forest reaches 99.95% accuracy (22 errors) with a train-test gap of only 0.016%.

## 2. Introduction

DDoS attacks are one of the most common threats on the Internet. Signature-based defences struggle with new attack patterns, so learning-based intrusion detection systems (IDS), which learn the statistical behaviour of flows (packet counts, packet sizes, inter-arrival times, TCP flags), are widely studied. The goal of this project is to detect DDoS flows from flow-level features alone, while avoiding features that make the problem artificially easy.

### 2.1 Literature survey

The CICIDS2017 dataset was released by the Canadian Institute for Cybersecurity (Sharafaldin et al., 2018) as a realistic replacement for older, outdated intrusion datasets such as KDD99. Flow features are extracted with CICFlowMeter and the data contains several attack families, including DDoS. Prior work on this dataset commonly applies tree-based ensembles (Random Forest, gradient boosting), KNN, SVM and neural networks, and typically reports very high accuracy on DDoS detection, since DDoS traffic is statistically very different from normal traffic. A recurring concern in the literature is that some features (for example the destination port or the initial TCP window size) can act as shortcuts specific to the capture environment, so they inflate results without helping real-world generalisation. We address this explicitly in our pipeline.

## 3. Resources

- **Data:** CICIDS2017, file `Friday-WorkingHours-Afternoon-DDos.pcap_ISCX.csv`.
- **Language and tools:** Python, Jupyter Notebook.
- **Libraries:** pandas, NumPy, Matplotlib, Seaborn, scikit-learn.

## 4. Work done

### Description of the data

The raw file has 225,745 rows and 79 columns: 78 numeric flow features and one text column, `Label`, with 128,027 `DDoS` and 97,718 `BENIGN` flows. Only `Flow Bytes/s` has missing values (4 rows).

### Pre-processing

1. Removed leading/trailing spaces from column names.
2. Dropped the 4 rows with missing values and 2,633 duplicate rows.
3. Dropped 10 constant columns (e.g. `Bwd PSH Flags`, `CWE Flag Count`, the bulk-transfer features).
4. Replaced ±∞ by NaN and dropped those rows. Final cleaned shape: 223,082 × 69.
5. Encoded the label: `BENIGN` = 0, `DDoS` = 1 (95,068 vs. 128,014 flows).

### Exploratory analysis

Box plots (log scale) of key features such as Flow Duration, Total Fwd Packets, Flow Bytes/s, Flow Packets/s, Packet Length Mean and Average Packet Size show clearly different distributions for benign and DDoS traffic. Spearman and Pearson correlations with the label were also computed; the strongest Spearman correlations were `Bwd Packet Length Min` (0.66), `URG Flag Count` (0.47) and `Bwd Packet Length Std` (0.46).

### Removing data leakage and multicollinearity

`Destination Port`, `Init_Win_bytes_forward` and `Init_Win_bytes_backward` were removed because they depend on the attack set-up rather than on the behaviour of the traffic (66 columns remain). Then 35 features with pairwise absolute correlation above 0.90 were dropped (e.g. subflow, idle, active, header-length and packet-length statistics), leaving 30 features plus the label.

### Train/test split and scaling

An 80/20 stratified split (random state 42) gives 178,465 training and 44,617 test samples (57.4% DDoS in training). Features were scaled with `RobustScaler` (fitted on the training set only) to reduce the effect of outliers.

### Models and results

Two classifiers were trained: KNN (k = 10) and Random Forest (100 trees, maximum depth 10).

| Model | Test accuracy | Misclassified flows | Notes |
|---|---|---|---|
| KNN (k = 10) | 99.63% | 167 / 44,617 | 118 FP, 49 FN; AUC ≈ 0.9996 |
| Random Forest | 99.95% | 22 / 44,617 | Train 99.966%, gap 0.016% |

*FP: benign flagged as DDoS, FN: DDoS missed.*

Both models achieve precision, recall and F1 of about 1.00 for both classes. The KNN confusion matrix (rows = actual, columns = predicted) is:

| | Predicted BENIGN | Predicted DDoS |
|---|---|---|
| **Actual BENIGN** | 18896 | 118 |
| **Actual DDoS** | 49 | 25554 |

The ROC curve for KNN (computed on a 5,000-sample subset for speed) has AUC 0.9996. Random Forest makes about 7.6 times fewer errors than KNN, and its train-test accuracy gap of 0.016% indicates no noticeable over-fitting.

### Feature importance

The most important Random Forest features are `Fwd Packet Length Max`, `Total Length of Fwd Packets`, `act_data_pkt_fwd`, `Bwd Packet Length Min` and `Total Fwd Packets`, i.e. mainly packet-size and packet-count features.

### Final architecture

Raw CSV → cleaning → leakage and collinearity removal → stratified 80/20 split → RobustScaler → KNN / Random Forest → evaluation (classification report, confusion matrix, ROC, feature importance).

### Code

GitHub location: https://github.com/kuldeepsinghyadav177-glitch/DDoS-Attack-Detection-using-Machine-Learning

## 5. Future work

- Report 5-fold stratified cross-validation (already coded in the notebook) to confirm the stability of the F1-score.
- Tune hyper-parameters (e.g. k, tree depth) and try gradient boosting or neural networks.
- Evaluate on the other CICIDS2017 days and attack types (multi-class detection) and test cross-dataset generalisation.
- Study real-time detection, i.e. latency and deployment of the model in an IDS.

## References

1. I. Sharafaldin, A. H. Lashkari, A. A. Ghorbani, "Toward Generating a New Intrusion Detection Dataset and Intrusion Traffic Characterization," ICISSP, 2018.
