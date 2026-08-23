# DDoS Attack Detection using Machine Learning

This project uses machine learning in a Jupyter Notebook to detect Distributed Denial of Service (DDoS) attacks from network traffic data.

## 1. Dataset
We used a part of the **CICIDS2017** dataset (Friday Afternoon Working Hours).
* **Total Records:** 73,929 rows
* **Features:** 79 columns
* **Labels:** `BENIGN` (Normal traffic) and `DDoS` (Attack traffic)
* **Data Split:** 55% DDoS, 45% Benign.

## 2. Project Steps

* **Step 1: Data Cleaning:** Removed extra spaces from column names, deleted duplicate rows, and handled missing or infinite values by dropping them.
* **Step 2: Feature Selection:** Dropped 10 columns that had the exact same value for all rows (zero variance). This reduced our features from 78 to 68.
* **Step 3: Data Analysis (EDA):** Used box plots and correlation checks to see the differences between normal and DDoS traffic.
* **Step 4: Data Preparation:** Split the data into 80% for training and 20% for testing. Used `StandardScaler` to normalize the data.
* **Step 5: Model Training:** Trained and tested two machine learning models:
  1. Logistic Regression
  2. Random Forest

## 3. Results

Here is how our models performed on the 20% test data:

## Model | Accuracy | F1-Score 
 **Logistic Regression** | 99.77% | 0.99 
 **Random Forest** | 99.98% | 0.99 


**Top 4 important features for detecting DDoS:**
1. `Avg Fwd Segment Size`
2. `Init_Win_bytes_forward`
3. `Fwd Packet Length Mean`
4. `Subflow Fwd Bytes`

## 4. How to Run This Project

**Requirements:**
Install the required Python libraries using this command:

pip install pandas numpy matplotlib seaborn scikit-learn notebook
