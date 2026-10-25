# DS607_Project
# Are U.S. Import Tariff Burdens Different Across Commodity Categories?

## 1. Project Overview

This project investigates changes in U.S. import tariff burdens following the 2025 trade policy changes. Using monthly U.S. import statistics from the U.S. Census Bureau, the analysis examines whether effective tariff rates increased across commodity categories and whether these changes were widespread or concentrated in particular categories.

The project uses statistical inference and resampling methods to evaluate changes in effective tariff rates between a pre-policy and post-policy period.

Rather than focusing solely on the overall aggregate tariff rate, this study examines differences across 96 HS-2 commodity chapters to understand how tariff changes were distributed across the U.S. import market.

### Research Questions

The main research question is:

**Did U.S. import tariff burdens change significantly across commodity categories following the 2025 trade policy changes?**

The supporting research questions are:

1. Did effective U.S. import tariff rates increase between the pre-policy and post-policy periods?
2. Were tariff increases widespread across commodity categories, or were they concentrated in specific categories?
3. How does the average change across individual commodity categories compare with the import-weighted aggregate change?
4. Is there an association between commodity groups and the magnitude of effective tariff rate changes?

---

## 2. Data Source

The dataset is obtained from the **U.S. Census Bureau's monthly U.S. import statistics**.

The data contains monthly import information at the Harmonized System (HS) 2-digit commodity chapter level.

- **Geographical coverage:** United States
- **Frequency:** Monthly
- **Unit of analysis:** HS-2 commodity chapters
- **Number of commodity chapters:** 96
- **Data format:** CSV

The raw dataset is stored in `data/census_import.csv`.

### Study Periods

The analysis compares two periods:

| Period | Date range |
|---|---|
| Pre-policy period | March 2024 – December 2024 |
| Post-policy period | March 2025 – December 2025 |

The analysis aggregates monthly import and calculated duty values within each period to obtain comparable effective tariff rates for each commodity chapter.

---

## 3. Variables and Data Dictionary

### Original variables

| Variable | Description |
|---|---|
| `date` | YYYY-MM of the observation |
| `I_COMMODITY` | HS-2 commodity chapter code |
| `I_COMMODITY_SDESC` | Short description of the commodity chapter |
| `GEN_VAL_MO` | General import value |
| `CAL_DUT_MO` | Calculated duties collected during the month |
| `CON_VAL_MO` | Consumption import customs value during the month |
| `DUT_VAL_MO` | Duitable import value |

### Derived variables

| Variable | Description |
|---|---|
| `etr_con` | Calculated duties divided by customs value |
| `etr_dut` | Calculated duties divided by dutiable value |
| `etr_pre` | Effective tariff rate during the pre-policy period |
| `etr_post` | Effective tariff rate during the post-policy period |

The main effective tariff rate measures are calculated as:

**Effective tariff rate based on customs value:**

$$
ETR_{con} = \frac{CAL\_DUT\_MO}{CON\_VAL\_MO}
$$

**Effective tariff rate based on dutiable value:**

$$
ETR_{dut} = \frac{CAL\_DUT\_MO}{DUT\_VAL\_MO}
$$

The change in effective tariff rate is calculated as:

$$
\Delta ETR_i = ETR_{post,i} - ETR_{pre,i}
$$

where $i$ represents an individual HS-2 commodity chapter.

A positive value indicates an increase in the effective tariff rate, while a negative value indicates a decrease.

---

## 4. Methodology

The project applies statistical inference to examine changes in effective tariff rates across commodity categories.

### 4.1 Exploratory Data Analysis (EDA)

Exploratory analysis is conducted to understand the structure and distribution of the import data.

This includes:

- Examining missing values and data consistency.
- Investigating monthly import and calculated duty trends.
- Comparing effective tariff rates across commodity categories.
- Examining the distribution of pre-policy and post-policy effective tariff rates.
- Identifying potential outliers and unusually large changes.

### 4.2 Chapter-Level Comparison

Each HS-2 commodity chapter is treated as an observation.

The pre-policy and post-policy effective tariff rates are calculated for each of the 96 commodity chapters.

The change in effective tariff rate is then calculated:

$$
\Delta ETR_i = ETR_{post,i} - ETR_{pre,i}
$$

The primary estimand is the average change across commodity chapters:

$$
\theta_A = \frac{1}{n}\sum_{i=1}^{n}\Delta ETR_i
$$

where:

- $n$ = number of commodity chapters (96).
- $\Delta ETR_i$ = change in effective tariff rate for chapter $i$.
- $\theta_A$ = average chapter-level change.

This gives each commodity chapter equal weight, regardless of its import size.

### 4.3 Confidence Interval Estimation

Several confidence interval approaches are used to quantify uncertainty surrounding the average chapter-level change.

These include:

- Classical Central Limit Theorem (CLT) confidence interval using the t-distribution.
- Bootstrap percentile confidence interval.
- Bootstrap basic confidence interval.
- Bootstrap-t confidence interval.

The bootstrap methods provide alternative ways to estimate uncertainty without relying entirely on the classical normal approximation.

### 4.4 Hypothesis Testing

Hypothesis tests are conducted to investigate whether the observed changes are statistically significant.

The analysis includes:

- Classical one-sample t-test.
- Sign test.
- Sign-flip permutation test.
- Wilcoxon signed-rank test.
- Bootstrap-based hypothesis testing.

The tests examine whether the changes in effective tariff rates are systematically positive rather than arising from random variation.

### 4.5 Import-Weighted Aggregate Comparison

In addition to the chapter-level analysis, the project calculates an aggregate effective tariff rate using total calculated duties relative to total customs value.

$$
ETR_{aggregate} =
\frac{\sum_i CAL\_DUT_i}
{\sum_i CON\_VAL_i}
$$

The aggregate change is then compared with the average chapter-level change.

This distinction is important because the two measures answer different questions:

- **Chapter-level average:** What is the average change experienced by a commodity chapter?
- **Import-weighted aggregate:** How much did the overall effective tariff burden change after accounting for differences in import values?

A large commodity category can have a much greater influence on the aggregate rate than a smaller category.

### 4.6 Chi-Square Test and Cramér's V

A chi-square test of independence is used to investigate the association between commodity groups and categories of effective tariff rate changes.

Cramér's V is calculated to measure the strength of the association.

A heatmap is also used to visualise the distribution of observations across commodity groups and effective tariff rate change categories.

---

## 5. Key Findings

The analysis produces several important findings.

### 5.1 Chapter-Level Changes

Across the 96 HS-2 commodity chapters:

- The average change in effective tariff rates was approximately **0.0673**.
- Approximately **99% of commodity chapters experienced positive changes**.
- The median change was approximately **0.0596**.

These findings suggest that the increase was broad-based rather than limited to a small number of commodity categories.

### 5.2 Confidence Intervals

The estimated 95% confidence intervals for the average chapter-level change were:

| Method | Lower bound | Upper bound |
|---|---:|---:|
| CLT (t-distribution) | 0.0571 | 0.0776 |
| Bootstrap percentile | 0.0577 | 0.0779 |
| Bootstrap basic | 0.0567 | 0.0770 |
| Bootstrap-t | 0.0580 | 0.0792 |

All four confidence intervals are above zero, providing evidence of a positive average change in effective tariff rates.

### 5.3 Hypothesis Testing

The statistical tests also provide evidence against the null hypothesis of no systematic increase.

| Test | p-value |
|---|---:|
| Sign-flip permutation test | 0.000050 |
| Sign test | 2.45 × 10⁻²⁷ |
| Bootstrap-t test | 0.000050 |
| Classical t-test | 6.69 × 10⁻²³ |
| Wilcoxon signed-rank test | 1.84 × 10⁻¹⁷ |

The results consistently indicate statistically significant positive changes across commodity chapters.

### 5.4 Aggregate vs Chapter-Level Comparison

The import-weighted aggregate change was approximately **0.0595**, compared with the unweighted chapter-level average of **0.0673**.

The difference highlights the importance of accounting for import size when interpreting the overall tariff burden.

The average chapter-level change is larger than the aggregate change, suggesting that smaller commodity categories may have experienced relatively larger increases than the categories carrying greater import values.

---

## 6. Repository Structure

The repository is organised as follows:

```text
DS607_Project/
│
├── data/
│   └── census_import.csv
│
├── outputs/
│   ├── figures/
│   │   └── (generated visualisations)
│   │
│   └── tables/
│       └── (generated statistical tables)
│
├── notebooks/
│   └── (analysis notebooks, if applicable)
│
└── README.md
```

- `data/`: Contains the original import dataset.
- `outputs/figures/`: Stores generated figures and visualisations.
- `outputs/tables/`: Stores statistical results, confidence intervals and summary tables.
- `notebooks/`: Contains the Python notebooks used for data processing and statistical analysis.
- `README.md`: Provides an overview of the project, data and methodology.

---

## 7. How to Run the Project

### Requirements

The analysis is conducted using Python.

Required libraries include:

```bash
pip install pandas numpy matplotlib seaborn scipy statsmodels
```

### Running the Analysis

1. Clone the repository:

```bash
git clone https://github.com/broccolixd/DS607_Project.git
```

2. Navigate to the project directory:

```bash
cd DS607_Project
```

3. Install the required Python libraries.

4. Open the analysis notebook in Jupyter Notebook or VS Code.

5. Run the notebook cells sequentially.

The notebook reads the dataset from the `data/` directory and saves generated figures and tables under `outputs/`.

---

## 8. Limitations

Several limitations should be considered when interpreting the findings:

1. **Effective tariff rates are not statutory tariff rates.** They represent calculated duties relative to import values and may be influenced by the composition of imports and the share of dutiable goods.

2. **Differences in import values:** The unweighted chapter-level average treats every commodity chapter equally, whereas the aggregate effective tariff rate reflects differences in import sizes.

3. **Temporal and economic factors:** Changes in effective tariff rates may also reflect changes in import composition, trade patterns, exemptions and other economic factors.

4. **Causal interpretation:** The pre-policy and post-policy comparison identifies changes over time but does not, by itself, establish that the policy changes caused the observed increases.

5. **Statistical dependence:** Commodity-level observations may not be completely independent, and the inference methods should be interpreted with this consideration in mind.

---

## 9. Conclusion

This project examines changes in U.S. import tariff burdens across 96 HS-2 commodity chapters using monthly import statistics.

The findings indicate a broad-based increase in effective tariff rates across commodity categories, supported by confidence interval estimation and multiple hypothesis testing approaches.

Comparing the unweighted chapter-level average with the import-weighted aggregate measure further demonstrates that the overall tariff burden and the average experience of an individual commodity chapter can differ.

Overall, the project illustrates how statistical inference, bootstrap methods and distributional analysis can be used to investigate changes in international trade patterns and tariff burdens.

---

## 10. Author and Course Information

**Course:** DS607 – Statistics for Data Science  
**Institution:** Singapore Management University (SMU)  
**Project:** Group Research Project  
**Data Source:** U.S. Census Bureau – Monthly U.S. Import Statistics

