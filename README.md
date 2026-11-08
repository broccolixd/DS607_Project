# Are U.S. Import Tariff Burdens Different Across Commodity Categories?

DS607 – Statistics for Data Science | Singapore Management University (SMU)

## 1. Project Overview

This project investigates changes in U.S. import tariff burdens following the 2025 trade policy changes. Using monthly U.S. import statistics from the U.S. Census Bureau, it examines whether effective tariff rates increased across commodity categories and whether the increase was widespread or concentrated in a few categories.

The analysis compares a pre-policy and a post-policy period using classical, resampling-based and Bayesian inference. Rather than focusing only on the overall aggregate tariff rate, it studies 96 HS-2 commodity chapters to understand how the change was distributed across the U.S. import market.

### Research Questions

**Main question:** Did U.S. import tariff burdens change significantly across commodity categories following the 2025 trade policy changes?

Supporting questions:

1. Did effective U.S. import tariff rates increase between the pre-policy and post-policy periods?
2. Were tariff increases widespread across commodity categories, or concentrated in specific ones?
3. How does the average change across individual commodity chapters compare with the import-weighted aggregate change?
4. Is there an association between commodity groups and the magnitude of effective tariff rate changes?

---

## 2. Data Source

The data comes from the **U.S. Census Bureau's monthly U.S. import statistics**, retrieved through the Census Data API (the collection code is in the appendix of the notebook; it was run once and the result saved as CSV).

- **Coverage:** United States, monthly, July 2023 – August 2026 (38 months)
- **Unit of analysis:** HS-2 commodity chapters
- **Chapters:** 98 in the raw file; chapters 98 and 99 (special classification and special import provisions) are excluded, leaving **96 chapters**
- **Format:** CSV, stored at `data/census_import.csv`

### Study Periods

| Period | Date range |
|---|---|
| Pre-policy | March 2024 – December 2024 |
| Post-policy | March 2025 – December 2025 |

Monthly values are summed within each period for each chapter, and effective tariff rates are computed from the totals. Chapters with pre-period imports below USD 10 million are dropped (this filter did not remove any of the 96 chapters).

---

## 3. Variables and Data Dictionary

### Original variables

| Variable | Description |
|---|---|
| `time` | Month of the observation (YYYY-MM) |
| `I_COMMODITY` | HS-2 commodity chapter code |
| `I_COMMODITY_SDESC` | Short description of the chapter |
| `GEN_VAL_MO` | General import value |
| `CON_VAL_MO` | Consumption import customs value |
| `DUT_VAL_MO` | Dutiable import value |
| `CAL_DUT_MO` | Calculated duties collected |

### Derived variables

| Variable | Description |
|---|---|
| `etr_con` | Calculated duties ÷ customs value (main measure) |
| `etr_dut` | Calculated duties ÷ dutiable value |
| `etr_con_pre`, `etr_con_post` | `etr_con` over the pre- and post-policy windows |
| `d_etr` | Change in effective tariff rate, post minus pre |

$$
ETR_{con} = \frac{CAL\_DUT}{CON\_VAL}, \qquad
ETR_{dut} = \frac{CAL\_DUT}{DUT\_VAL}
$$

$$
\Delta ETR_i = ETR_{post,i} - ETR_{pre,i}
$$

where $i$ indexes an HS-2 chapter. A positive value means the effective rate rose. Rates are stored as fractions, so a change of 0.0837 equals 8.4 percentage points.

---

## 4. Methodology

### 4.1 Exploratory Data Analysis

- Checking missing values, duplicate chapter-months and data consistency.
- Monthly aggregate effective tariff rate over time.
- Distribution of chapter-level changes (histogram, boxplot, ECDF) and outlier identification.
- Concentration analysis: how much of the total duty increase comes from the largest chapters.
- Association between import size, pre-period rate and the size of the change (Pearson and Kendall's tau with bootstrap intervals).

### 4.2 Chapter-Level Comparison

Each chapter is one observation. The primary estimand is the unweighted average change, which gives every chapter equal weight regardless of import size:

$$
\theta_A = \frac{1}{n}\sum_{i=1}^{n}\Delta ETR_i, \qquad n = 96
$$

### 4.3 Frequentist and Bootstrap Confidence Intervals

95% intervals for $\theta_A$:

- Classical CLT interval using the t-distribution
- Bootstrap percentile interval
- Bootstrap basic interval
- Bootstrap-t interval

### 4.4 Bayesian Credible Interval

To complement the frequentist intervals, the mean chapter change $\mu$ is also estimated in a Bayesian framework.

**Model.** Chapter changes are treated as $\Delta ETR_i \sim N(\mu, \sigma^2)$ with a conjugate Normal-Inverse-Gamma prior:

$$
\mu \mid \sigma^2 \sim N\!\left(\mu_0, \frac{\sigma^2}{\kappa_0}\right), \qquad
\sigma^2 \sim \text{Inv-Gamma}(a_0, b_0)
$$

The posterior is available in closed form, and the marginal posterior of $\mu$ is a Student-t distribution with $2a_n$ degrees of freedom, location $\mu_n$ and scale $\sqrt{b_n / (a_n \kappa_n)}$, where

$$
\kappa_n = \kappa_0 + n, \quad
\mu_n = \frac{\kappa_0\mu_0 + n\bar{x}}{\kappa_n}, \quad
a_n = a_0 + \frac{n}{2}, \quad
b_n = b_0 + \tfrac{1}{2}\sum_i (x_i-\bar{x})^2 + \frac{\kappa_0 n(\bar{x}-\mu_0)^2}{2\kappa_n}
$$

The **95% credible interval** is the central 2.5%–97.5% range of this posterior. Three priors are compared to check how sensitive the result is to prior choice:

| Prior | $\mu_0$ | $\kappa_0$ | $a_0$ | $b_0$ | Description |
|---|---:|---:|---:|---:|---|
| Reference (flat) | 0 | 0 | −0.5 | 0 | Non-informative; reproduces the classical t-interval |
| Weak, centred at 0 | 0 | 1 | 1 | 0.0025 | Worth about one observation at zero |
| Sceptical | 0 | 20 | 1 | 0.0025 | Worth about 20 observations centred at zero (no change) |

Besides the interval, the posterior probabilities $P(\mu > 0)$ and $P(\mu > 0.05)$ are reported, where 0.05 (5 percentage points) is used as the threshold for a practically meaningful increase.

### 4.5 Hypothesis Testing

- Classical one-sample t-test
- Sign test (median)
- Sign-flip permutation test (mean)
- Wilcoxon signed-rank test
- Bootstrap-t test (mean)

### 4.6 Import-Weighted Aggregate Comparison

The aggregate effective rate uses total duties relative to total customs value, so large chapters carry more weight:

$$
ETR_{aggregate} = \frac{\sum_i CAL\_DUT_i}{\sum_i CON\_VAL_i}
$$

Its change is estimated with a jackknife Wald interval and percentile, basic and bootstrap-t intervals, and compared with the unweighted chapter-level average.

- **Chapter-level average:** the change experienced by the typical commodity chapter.
- **Import-weighted aggregate:** the change in the overall tariff burden on total imports.

### 4.7 Chi-Square Test and Cramér's V

Chapters are placed in five commodity groups (agriculture and food; minerals, chemicals, plastics and rubber; leather, wood, paper, textiles and footwear; stone, glass, jewellery and base metals; machinery, transport, instruments and miscellaneous) and the change is cut into terciles (Low, Mid, High). A chi-square test of independence and Cramér's V measure the association, and a stacked bar chart shows the distribution.

### 4.8 Reproducibility

All random procedures (bootstrap, permutation, Kendall's tau bootstrap) use a fixed seed (`SEED = 607`) set at the top of the notebook, so results are reproducible.

---

## 5. Key Findings

### 5.1 Chapter-Level Changes

Across the 96 HS-2 chapters:

- The mean change in effective tariff rate was **0.0837** (about 8.4 percentage points).
- The median change was **0.0786**, close to the mean, so outliers are not driving the result.
- **95 of 96 chapters (99%)** saw an increase. The only decrease was Mineral fuel, oil etc. (−0.0005, effectively zero).
- The distribution is moderately right-skewed (skewness 1.12, excess kurtosis 1.93). Four upper outliers were flagged: umbrellas and walking-sticks, prepared feathers and artificial flowers, aluminium, and iron and steel articles (the last two rose by about 27–29 points).

The increase was broad-based rather than limited to a few chapters.

### 5.2 Confidence Intervals (frequentist)

95% intervals for the mean chapter-level change:

| Method | Lower | Upper | Width |
|---|---:|---:|---:|
| CLT (t-distribution) | 0.0721 | 0.0952 | 0.0231 |
| Bootstrap percentile | 0.0728 | 0.0953 | 0.0225 |
| Bootstrap basic | 0.0720 | 0.0945 | 0.0225 |
| Bootstrap-t | 0.0730 | 0.0964 | 0.0235 |

All four intervals are nearly identical and lie well above zero, so the result does not depend on a normality assumption.

### 5.3 Bayesian Credible Interval

95% credible intervals for the mean chapter-level change $\mu$:

| Prior | Posterior mean | 95% credible interval | $P(\mu>0)$ | $P(\mu>0.05)$ |
|---|---:|---:|---:|---:|
| Reference (flat) | 0.0837 | [0.0721, 0.0952] | 1.0000 | 1.0000 |
| Weak, centred at 0 | 0.0828 | [0.0713, 0.0943] | 1.0000 | 1.0000 |
| Sceptical (20 obs at 0) | 0.0693 | [0.0571, 0.0814] | 1.0000 | 0.9989 |

Interpretation:

- With the flat reference prior, the credible interval is identical to the classical t-interval [0.0721, 0.0952], as expected. The Bayesian reading is the direct one: given the data and the model, there is a 95% posterior probability that the true mean change lies in this range.
- A weak prior centred at zero barely moves the result.
- Even a sceptical prior, worth 20 observations of "no change", pulls the posterior mean down only to 0.0693. The credible interval stays well above zero and above the 0.05 threshold, with $P(\mu>0.05) = 0.9989$.
- The conclusion that the average chapter-level increase is positive and substantial does not depend on prior choice.

The posterior density (weak prior) with its credible interval shaded is plotted in the notebook and saved in `outputs/figures/`.

### 5.4 Hypothesis Tests

| Test | p-value |
|---|---:|
| Sign-flip permutation test | 5.0 × 10⁻⁵ (< 0.0001) |
| Bootstrap-t test | 5.0 × 10⁻⁵ (< 0.0001) |
| Sign test | 2.45 × 10⁻²⁷ |
| Classical t-test | 1.30 × 10⁻²⁵ |
| Wilcoxon signed-rank test | 1.84 × 10⁻¹⁷ |

All five tests reject "no change". The two resampling p-values are the smallest the procedure can return (1/(B+1) with B = 20,000), so they are reported as p < 0.0001 rather than exact values.

### 5.5 Aggregate vs Chapter-Level Comparison

| Measure | Change | 95% interval (percentile) |
|---|---:|---:|
| Unweighted chapter mean | 0.0837 | [0.0728, 0.0953] |
| Import-weighted aggregate | 0.0682 | [0.0488, 0.0947] |

Other aggregate intervals: Wald (jackknife) [0.0442, 0.0921], basic [0.0416, 0.0876], bootstrap-t [0.0424, 0.0978].

- The aggregate change is **lower** than the unweighted mean. Large chapters such as machinery (ch. 84, +4.8 points) and electrical equipment (ch. 85, +6.9 points) rose by less than the typical small chapter, while metals, toys and apparel rose more but carry less trade.
- The aggregate intervals are about twice as wide (roughly 0.046–0.055 against 0.023), because a few large chapters dominate the total.
- The top 5 chapters account for about 61% of the net increase in duties collected and the top 10 for about 75%, while those 10 chapters hold about 58% of pre-period imports.

### 5.6 Association with Commodity Group

- Pre-period import size is almost unrelated to the size of the change (Pearson r = −0.08, Kendall's tau = −0.08, p = 0.24).
- Chapters with a higher pre-period effective rate tended to see larger increases (Kendall's tau = 0.48).
- Chi-square test of commodity group against change tercile: $\chi^2 = 30.67$, df = 8, p = 0.0002, **Cramér's V = 0.40**, a moderately strong association.
- The increase was concentrated in *Leather, wood, paper, textiles and footwear* (17 of 27 chapters in the High tercile) and weakest in *Agriculture and food* (1 of 24 in the High tercile).

---

## 6. Repository Structure

```text
DS607_Project/
│
├── README.md                  Project overview, methods and findings
├── AI_USAGE.md                Disclosure of how AI tools were used
├── CONTRIBUTIONS.md           Contribution of each team member
├── requirements.txt           Python dependencies (or environment.yml)
│
├── data/
│   └── census_import.csv      Raw Census import data
│
├── notebooks/
│   └── DS607_Project.ipynb    Full analysis (data prep, inference, Bayesian, tests)
│
└── outputs/
    ├── figures/               Generated figures
    ├── tables/                Generated tables (intervals, tests, Bayesian results)
    └── presentation/          Presentation PDF
```

---

## 7. How to Run the Project

### Requirements

Python 3.10 or later. Install the dependencies:

```bash
pip install -r requirements.txt
```

The main libraries are pandas, numpy, matplotlib, seaborn and scipy (`requests` is only needed for the data-collection appendix).

### Running the Analysis

1. Clone the repository:

   ```bash
   git clone https://github.com/broccolixd/DS607_Project.git
   cd DS607_Project
   ```

2. Install the requirements (above).
3. Open `notebooks/DS607_Project.ipynb` in Jupyter Notebook or VS Code.
4. Run all cells from top to bottom.

The notebook reads `data/census_import.csv` using a relative path and writes figures and tables to `outputs/figures/` and `outputs/tables/`. Random seeds are fixed, so results are reproducible. The Appendix cells that call the Census API require a Census API key and are not needed to reproduce the analysis, since the downloaded data is included.

---

## 8. Limitations

1. **Effective rates are not statutory rates.** They are duties collected relative to import value and can change when the product or country mix changes.
2. **Before/after design with no control group.** The comparison describes the change but does not show that the policy caused it.
3. **HS-2 chapters are broad** and can hide large differences between individual products.
4. **Window-specific result.** The post window ends in December 2025, and the aggregate rate fell in 2026, so the findings apply to this window only.
5. **Dependence between chapters.** Inference treats chapters as independent draws, but the 96 chapters are the full set of goods and related chapters move together. Very small p-values should be read as a sign of a large effect rather than exact probabilities.
6. **Bayesian model assumptions.** The credible interval assumes approximately normal chapter changes. The data are right-skewed with heavy tails, which is why it is read alongside the bootstrap intervals; the priors are illustrative and were chosen to test sensitivity, not elicited from experts.
7. **Judgement calls.** The USD 10 million import filter, the commodity group boundaries, the tercile cut and the 0.05 "meaningful increase" threshold were not tested for sensitivity. Chi-square expected counts in the smallest group fall slightly below 5, and binning discards information (a Kruskal-Wallis test on the raw changes is a natural follow-up).

---

## 9. Conclusion

Effective U.S. import tariff rates rose across almost every HS-2 chapter between March–December 2024 and March–December 2025: by about 8.4 percentage points for the average chapter and 6.8 points for the import-weighted aggregate. The result is consistent across the t-interval, three bootstrap intervals, five hypothesis tests and Bayesian credible intervals under flat, weak and sceptical priors, and it is largest in textiles, leather, wood and paper goods and smallest in agriculture and food. The gap between the chapter-level and aggregate figures shows that the burden on the typical chapter and the burden on total imports are different quantities.

---

## 10. Course Information

**Course:** DS607 – Statistics for Data Science
**Institution:** Singapore Management University (SMU)
**Project:** Group Research Project
**Data source:** U.S. Census Bureau – Monthly U.S. Import Statistics