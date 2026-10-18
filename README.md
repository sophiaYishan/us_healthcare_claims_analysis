# Healthcare Claims Analytics: Cost Drivers & Provider Fraud Risk

Payer-perspective analysis of US Medicare claims: where claims cost concentrates, what drives it, and which billing patterns are associated with providers flagged as potential fraud.

## Business Questions
From a health insurer's (payer's) perspective:
1. Where does claims cost concentrate, and what drives it?
2. Which providers show billing patterns associated with potential fraud?
3. How could a claims team prioritise provider reviews?

## Data
- Source: [Healthcare Provider Fraud Detection Analysis (Kaggle)](https://www.kaggle.com/datasets/rohitrox/healthcare-provider-fraud-detection-analysis)
- US Medicare claims ending in calendar year 2009: 40,474 inpatient and 517,737 outpatient claims, 138,556 beneficiaries, and 5,410 providers with a provider-level potential-fraud flag
- Raw data is not included in this repo; download it from the link above
- Field definitions, data checks and derived fields: see the [Data Dictionary](data_dictionary.md)

## Key Findings

**Where the money goes**
- **Inpatient stays drive cost:** inpatient claims are 7% of claims but 73% of reimbursement ($408M of $557M).
- **Cost is highly concentrated:** the top 1% of beneficiaries account for 16% of reimbursement, the top 5% for 44%, and the top 20% for 81%. A care management program for the top 5% would reach 1 in 20 members but address over 40% of spending.
- **Claim amounts are skewed:** the mean outpatient claim ($298) is 3.7 times the median ($80), so this analysis uses medians and percentiles rather than averages.

**What drives it**
- **Chronic conditions:** median annual cost per beneficiary rises from $170 with no chronic conditions to about $8,000 with 10.
- **Admissions are the mechanism:** the share of beneficiaries admitted at least once rises from 3% with no chronic conditions to 62% with 10. Preventing avoidable admissions among members with many chronic conditions is the main cost lever.
- **Kidney disease** is associated with the highest cost: median cost is 4.8 times that of beneficiaries without it, followed by COPD (4.1×) and stroke (3.7×). Conditions often occur together, so these ratios reflect each condition plus its typical co-occurring conditions.

**Provider fraud risk**
- **9.35% of providers** are flagged as potential fraud, but their claims make up **38% of all claims**.
- **Flagged providers are much larger:** median 156 vs 27 claims (5.8×) and $373K vs $15K reimbursed (25×), and they are inpatient-heavy (median inpatient share of cost 91% vs 0%). Billing intensity differs far less: 1.19 vs 1.08 claims per beneficiary. Much of the difference reflects provider size and type, so review rules should compare providers against peers rather than flag large hospitals outright.

**Payment integrity checks**
- **Claims billed after discharge:** 17 inpatient claims, each from a different provider, end after the patient's discharge date. All 17 providers are flagged as potential fraud. The rule is highly precise but narrow: it identifies 17 of 506 flagged providers (3.4%).
- **High-dollar outpatient claims:** 7 outpatient claims exceed $20K (about 6 times the outpatient 99th percentile of $3,300). 5 of 7 (71%) come from flagged providers, roughly twice the 37% rate across all outpatient claims. Together they were paid $637,290.

## Data Quality Notes
- Chronic condition fields are coded 1/2 with no codebook. 1 = Yes was inferred from three independent checks: prevalence, cost, and consistency with the end-stage renal disease indicator.
- 3.7% of claims have zero reimbursement and none are negative. About 90% of zero-paid claims also have no deductible, suggesting denied or no-payment claims.
- The beneficiary table's annual reimbursement fields do not reconcile with the claims: 26% of beneficiaries have annual inpatient reimbursement, but only 23% have an inpatient claim. All cost analysis therefore uses claim-level amounts.

## Approach
- [x] Data understanding and validation (keys, joins, missing values, undocumented codes)
- [x] Cleaning and integration into one claim-level table and one provider-level table
- [x] Descriptive analysis: cost concentration, cost drivers, provider patterns
- [x] Rule-based payment integrity checks
- [ ] Provider-level fraud risk model
- [ ] Singapore context: MOH hospital bill benchmarks

## Next Steps: From Findings to Review Policy
Payment integrity teams typically combine three tiers:

| Tier | What it is | Evidence so far | Action |
|---|---|---|---|
| 1. Hard rules | Claims that are wrong by definition | Billed after discharge: 17 of 17 providers flagged | Pre-payment check |
| 2. Outlier triggers | Unusual compared with peer providers | High-dollar outpatient claims: 5 of 7 flagged | Post-payment review |
| 3. Risk score | Combines many weak signals | Provider-level model (next) | Prioritise the review queue |

Next: define peer groups (e.g. inpatient-heavy vs outpatient-only providers), test outlier rules for precision and coverage against the 9.35% base rate, then build a provider-level model evaluated on recall at a fixed review capacity rather than accuracy.

## Limitations
- Fraud labels are provider-level and unconfirmed ("potential"); how they were created is not documented.
- One year of data, so no trends or year-on-year analysis.
- US Medicare coding and payment rules differ from Singapore's (e.g. ICD-9-CM vs ICD-10-AM, and the MediShield Life / Integrated Shield structure).

## Repository Structure
```
README.md
data_dictionary.md        Field definitions, data checks, derived fields
claim_analysis.ipynb      Data validation, cleaning, provider table, analyses
```

## Tools
Python (pandas, NumPy, matplotlib, seaborn)

## Notes
- Providers are described as "flagged as potential fraud": the labels are unconfirmed.
- Race is excluded from fraud comparisons and modelling.
