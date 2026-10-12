# Healthcare Claims Analytics: Cost Drivers & Provider Fraud Risk

## Business Question
From a health insurer's (payer's) perspective:
1. Where does claims cost concentrate, and what drives it?
2. Which providers show billing patterns associated with potential fraud?
3. How could a claims team prioritise provider reviews?

## Data
- Source: [Healthcare Provider Fraud Detection Analysis (Kaggle)](https://www.kaggle.com/datasets/rohitrox/healthcare-provider-fraud-detection-analysis)
- US Medicare claims: 40,474 inpatient and 517,737 outpatient claims, 138,556 beneficiaries, and 5,410 providers with a provider-level potential-fraud flag
- Raw data is not included in this repo; download it from the link above
- Field definitions and data checks: see the [Data Dictionary](data_dictionary.md)

## Approach
- [x] Data understanding and validation (keys, joins, missing values, undocumented codes)
- [x] Cleaning and integration into one claim-level table
- [ ] Descriptive analysis: cost concentration, cost drivers, provider patterns
- [ ] Provider-level fraud risk model
- [ ] Singapore context: MOH hospital bill benchmarks

## Progress So Far

**Data validation**
- Primary keys are unique in all four tables; every claim links to a beneficiary and to a labelled provider.
- Chronic condition fields are coded 1/2 with no codebook. The meaning (1 = Yes) was inferred from three independent checks: prevalence, cost, and consistency with the ESRD indicator.
- 3.7% of claims have zero reimbursement and none are negative. About 90% of the zero-paid claims also have no deductible, which suggests denied or no-payment claims.
- For 49 inpatient claims, the claim period does not match the hospital stay; flagged for further review.

**First finding**
- **9.35% of providers** are flagged as potential fraud, yet their claims make up **38.12% of all claims**. Flagged providers bill far more claims per provider than the rest.

## Key Findings
_To be added after the descriptive analysis._

## Tools
Python (pandas, matplotlib/seaborn, scikit-learn)

## Notes
- Providers are described as "flagged as potential fraud": the labels are unconfirmed.
- Race is excluded from fraud comparisons and modelling.
