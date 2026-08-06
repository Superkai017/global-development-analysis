# global-development-analysis
# Country Socioeconomic & Health Profiling

This repository contains a global development dataset (`Country-data.csv`) encompassing 167 countries with key socio-economic and health indicators. The dataset is designed for unsupervised machine learning tasks, such as K-Means or Hierarchical Clustering, to group countries based on development status and allocate international aid or policy resources effectively.

---

## Dataset Overview
* **data_url:** https://www.kaggle.com/datasets/rohan0301/unsupervised-learning-on-country-data/data
* **Total Records:** 167 countries
* **Total Features:** 10 variables (1 categorical identifier, 9 numerical metrics)
* **Missing Values:** None (100% complete across all 167 rows)

### Data Dictionary

| Feature Column | Data Type | Description | Range / Summary (Min - Max) |
| :--- | :--- | :--- | :--- |
| `country` | String | Name of the country | 167 unique countries |
| `child_mort` | Float | Death of children under 5 years of age per 1,000 live births | 2.60 – 208.00 |
| `exports` | Float | Exports of goods/services per capita (as % of GDP per capita) | 0.11% – 200.00% |
| `health` | Float | Total health spending per capita (as % of GDP per capita) | 1.81% – 17.90% |
| `imports` | Float | Imports of goods/services per capita (as % of GDP per capita) | 0.07% – 174.00% |
| `income` | Integer | Net income per person (USD) | $609 – $125,000 |
| `inflation` | Float | Measurement of the annual growth rate of the Total GDP | -4.21% – 104.00% |
| `life_expec` | Float | Average number of years a newborn child would live if current mortality patterns remain constant | 32.10 – 82.80 years |
| `total_fer` | Float | Average number of children born to a woman if current age-fertility rates remain constant | 1.15 – 7.49 children |
| `gdpp` | Integer | Gross Domestic Product (GDP) per capita (USD) | $231 – $105,000 |

---

## Repository Structure

```text
├── data/
│   └── Country-data.csv       # Main dataset file
├── notebooks/
│   └── country_clustering.ipynb # EDA, scaling, and clustering analysis
├── src/
│   ├── data_processing.py    # Preprocessing & scaling module
│   └── modeling.py           # Clustering algorithm models
├── .gitignore
├── README.md
└── requirements.txt
