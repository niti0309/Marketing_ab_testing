# Marketing A/B Test Analysis

An end-to-end analysis of a large-scale marketing A/B test, examining whether showing users an **ad** (vs. a **PSA / public service announcement** control) increases conversion rate, using statistical testing, segmentation, and a Multi-Armed Bandit simulation as a comparison to the fixed-split design actually used.

## Business Question

Does the ad campaign causally increase conversions, or would users have converted anyway? The dataset uses a PSA as the control group (rather than "no ad") specifically to isolate the ad's incremental effect from general exposure/attention effects.

## Dataset

[Marketing A/B Testing (faviovaz)](https://www.kaggle.com/datasets/faviovaz/marketing-ab-testing), Kaggle

- **588,101 users**, naturally imbalanced 96% ad / 4% PSA split
- Columns: `user_id`, `test_group` (ad/psa), `converted` (bool), `total_ads`, `most_ads_day`, `most_ads_hour`
- Conversion event and product category are anonymized in the source data

## Methodology

1. **Data cleaning**: column standardization, null/duplicate checks, outlier flagging (99.9th percentile of ad exposure)
2. **Experiment framing**: explicit null/alternative hypotheses, primary metric (conversion rate), guardrail metric (ad exposure), α = 0.05
3. **Exploratory analysis**: conversion rate by group, by day of week, by hour, exposure distribution
4. **Statistical testing**
   - Two-proportion z-test
   - 95% confidence intervals (normal approximation)
   - Statistical power analysis (post-hoc power + required sample size for 80% power)
   - Bootstrap resampling (5,000 iterations) as an independent validation of the CI
5. **Segmentation**: dose-response analysis (conversion vs. ad exposure decile), time-of-day/day-of-week effects
6. **Multi-Armed Bandit simulation**: Thompson Sampling simulation benchmarking adaptive traffic allocation against the fixed 96/4 split actually used, including cumulative regret and convergence analysis

## Key Results

| Metric | Ad group | PSA group |
|---|---|---|
| Sample size | 564,577 | 23,524 |
| Conversion rate | 2.55% | 1.79% |
| 95% CI | (2.51%, 2.60%) | (1.62%, 1.95%) |

- **Relative lift: +43.1%** (ad vs. PSA)
- **z = 7.37, p < 0.0001**: highly statistically significant
- **Statistical power achieved: ~100%**, well beyond what was needed to detect this effect
- **Bootstrap 95% CI for the difference: [0.59%, 0.94%]**, confirming significance independently of the z-test
- Clear **dose-response relationship**: conversion increases with ad exposure, with diminishing returns at higher exposure levels
- **Bandit simulation finding:** Thompson Sampling sent *more* traffic to the losing arm (11.3%) during its exploration phase than the fixed split actually used (4.0%), showing that a well-chosen fixed split can already be near-optimal, and bandits aren't a universal upgrade over traditional A/B testing

## Business Recommendation

Continue running ads over PSAs. At current traffic volume (~565,000 ad-exposed users), the observed lift translates to roughly **4,300 additional conversions** compared to a PSA-only approach. Given the strong dose-response pattern with diminishing returns, there's an opportunity to investigate optimal ad frequency for budget efficiency in a follow-up analysis.

## Repo Structure

```
├── marketing_ab_test_analysis.ipynb   # Full analysis notebook
├── marketing_AB.csv                   # Dataset (download from Kaggle, not included in repo)
└── README.md
```

## Tools

Python · pandas · numpy · matplotlib · seaborn · statsmodels

## How to Run

1. Download `marketing_AB.csv` from [Kaggle](https://www.kaggle.com/datasets/faviovaz/marketing-ab-testing)
2. Place it in the repo root
3. `pip install pandas numpy matplotlib seaborn statsmodels`
4. Open `marketing_ab_test_analysis.ipynb` in Jupyter and run all cells
