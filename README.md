# Machine Learning-Based Intrusion Detection in RT-IoT Network Traffic

Using supervised machine learning to detect whether an IoT network transmission is under attack, based on its network traffic signals. Trained on the RT-IoT2022 dataset (123,117 flows).

*Research poster presented at the Stanford Pre-Collegiate Summer Institutes symposium (Introduction to Machine Learning, June 2026).*

![Research poster](images/poster.jpg)

## The Problem
As more everyday actions run over networks, from uploading a file to controlling a smart light bulb, network security matters more. IoT devices are especially easy targets. This project asks: **can a machine learning model tell normal IoT traffic apart from an attack, using only traffic features like packet rates, sizes, and timing?**

## Dataset
[RT-IoT2022](https://archive.ics.uci.edu/dataset/942/rt-iot2022) (UCI Machine Learning Repository), created by B. S. Sharmila and Rohini Nagapadma (2023).

- **123,117 instances, 83 features**, captured with the Zeek network monitor and Flowmeter plugin
- **Normal traffic** from ThingSpeak-LED, Wipro-Bulb, and MQTT-Temp IoT devices
- **Attacks:** Brute-Force SSH, DDoS (Hping and Slowloris), ARP poisoning, and Nmap scans (FIN, OS detection, TCP, UDP, XMAS tree)

I chose this dataset because its traffic is realistic and large, which makes it well suited for training intrusion detection models.

## How It Works

**1. Data cleaning**
- Removed randomized values that carried no meaning
- One-hot encoded the `proto` (TCP/UDP) and `service` (DNS, HTTP, MQTT, NTP, SSL) columns
- Recoded the label as binary: **0 = Normal** (MQTT Publish, ThingSpeak, Wipro Bulb) and **1 = Attack** (all attack types)

**2. Feature selection with Phi-K correlation**
- Ran a Phi-K correlation matrix and dropped features with correlation below 0.1 to the label
- This narrowed ~90 features down to **34**, covering packet rates, sizes, timing patterns, protocol/service types, and TCP flag activity

<img src="images/phik_correlation.png" width="450" alt="Phi-K correlation matrix">

**3. Models**
| Type | Models |
|------|--------|
| Classification | Decision Tree, Tuned Decision Tree, Bagging Classifier, Tuned Bagging Classifier, Random Forest, Tuned Random Forest |
| Regression | Logistic Regression |
| Ensemble | XGBoost, Tuned XGBoost |

**4. Evaluation**
Compared accuracy, recall, precision, and F1 on training and test sets, plus an ROC curve for logistic regression and feature importance for the tree models. The best model was chosen for **high recall** (catching as many attacks as possible) and **no overfitting** (small gap between training and test scores).

## Results

| Model | Test Accuracy | Test Recall | Test Precision | Test F1 |
|-------|:---:|:---:|:---:|:---:|
| **Tuned XGBoost** | **0.998** | **0.984** | **0.993** | **0.988** |
| XGBoost | 0.998 | 0.987 | 0.997 | 0.992 |
| Random Forest | 0.998 | 0.986 | 0.995 | 0.991 |
| Tuned Bagging Classifier | 0.998 | 0.986 | 0.995 | 0.990 |
| Bagging Classifier | 0.998 | 0.983 | 0.995 | 0.989 |
| Decision Tree | 0.996 | 0.989 | 0.975 | 0.982 |
| Tuned Random Forest | 0.996 | 0.986 | 0.972 | 0.979 |
| Tuned Decision Tree | 0.986 | 0.993 | 0.886 | 0.936 |
| Logistic Regression | 0.984 | 0.953 | 0.893 | 0.922 |

Full train vs. test scores: [images/results_table.png](images/results_table.png)

**Why Tuned XGBoost?** It had a very small gap between training and test scores while keeping all metrics high. The untuned XGBoost scored slightly higher, but its 0.999 training accuracy suggested possible overfitting.

<p>
<img src="images/roc_curve.png" width="420" alt="ROC curve, AUC 0.9956">
<img src="images/feature_importance_xgboost.png" width="420" alt="XGBoost feature importance">
</p>

- Logistic regression reached an **ROC AUC of 0.9956**, nearly perfect even for a simple linear model.
- XGBoost handled non-linear relationships in the traffic that logistic regression can't capture.

## Key Finding
Across the best models, the two most important signals of an attack were:
1. **`fwd_iat.max`**: maximum inter-arrival time between consecutive forward packets
2. **`flow_iat.max`**: maximum inter-arrival time between packets in the flow

In other words, **unusual timing gaps between packets** are the strongest warning sign of an intrusion. Monitoring these could help make real-time IoT networks safer.

## Tech Used
Python · pandas · scikit-learn · XGBoost · phik · matplotlib · seaborn · Jupyter

## How to Run
```bash
git clone https://github.com/emyms29/RT-IoT.git
cd RT-IoT
pip install -r requirements.txt
jupyter notebook
```
Then open `ProjectEMa.ipynb` and run all cells.

## References
- B. S. Sharmila and R. Nagapadma. "RT-IoT2022," UCI Machine Learning Repository, 2023. https://doi.org/10.24432/C5P338
- OpenAI, "ChatGPT," 2026. https://chat.openai.com/

## Author
Emily (Yiming) Ma · Stanford Pre-Collegiate Summer Institutes · [LinkedIn](https://www.linkedin.com/in/yiming-ma-06901a389/)
