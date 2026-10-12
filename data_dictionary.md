# Data Dictionary

Source: [Healthcare Provider Fraud Detection Analysis (Kaggle)](https://www.kaggle.com/datasets/rohitrox/healthcare-provider-fraud-detection-analysis). Only the four **Train** files are used.

## Table overview

| Table | File | Grain (one row per) | Rows × columns | Primary key | Joins to |
|---|---|---|---|---|---|
| Labels | `Train-*.csv` | Provider | 5,410 × 2 | `Provider` | Inpatient, Outpatient (on `Provider`) |
| Beneficiary | `Train_Beneficiarydata-*.csv` | Beneficiary (patient) | 138,556 × 25 | `BeneID` | Inpatient, Outpatient (on `BeneID`) |
| Inpatient | `Train_Inpatientdata-*.csv` | Inpatient claim | 40,474 × 30 | `ClaimID` | Beneficiary, Labels |
| Outpatient | `Train_Outpatientdata-*.csv` | Outpatient claim | 517,737 × 27 | `ClaimID` | Beneficiary, Labels |

**Key and join checks:**
- Primary keys are unique in all four tables (0 duplicates).
- No `ClaimID` overlap between inpatient and outpatient claims.
- Every `BeneID` in the claims tables exists in Beneficiary, and every beneficiary has at least one claim (138,556 distinct claim BeneIDs = 138,556 beneficiaries).
- The providers in the claims tables match the 5,410 providers in Labels exactly.

---

## 1. Labels (`Train-*.csv`)

| Column | Meaning | Notes |
|---|---|---|
| `Provider` | Healthcare provider (hospital / practice) ID | Join key to claims |
| `PotentialFraud` | Whether the provider is flagged as potential fraud (Yes/No) | Target variable. The label is at **provider level**, not claim level, and is unconfirmed ("potential"). 506 Yes / 4,904 No (9.4% flagged) |

---

## 2. Beneficiary (`Train_Beneficiarydata-*.csv`)

| Column | Meaning | Notes |
|---|---|---|
| `BeneID` | Beneficiary (patient) ID | Join key to claims. No nulls |
| `DOB` | Date of birth | No nulls |
| `DOD` | Date of death | 99.0% null; null = alive |
| `Gender` | Gender (coded) | Values 1 (59,450) and 2 (79,106). Mapping not documented in the source |
| `Race` | Race (coded) | Protected attribute; excluded from fraud comparisons and modelling |
| `RenalDiseaseIndicator` | End-stage renal disease (ESRD) flag | Stored as text: `'0'` (118,978) and `'Y'` (19,578; 14.1%) |
| `State` | State code | Numeric code, not state name |
| `County` | County code | Numeric code |
| `NoOfMonths_PartACov` | Months of Medicare Part A (hospital) coverage in the year | No nulls |
| `NoOfMonths_PartBCov` | Months of Medicare Part B (medical/outpatient) coverage in the year | No nulls |
| `ChronicCond_Alzheimer` | Alzheimer's flag | 1 = Yes, 0 = No (see coding note). Prevalence 33% |
| `ChronicCond_Heartfailure` | Heart failure flag | Prevalence 49% |
| `ChronicCond_KidneyDisease` | Kidney disease flag | Prevalence 31% |
| `ChronicCond_Cancer` | Cancer flag | Prevalence 12% |
| `ChronicCond_ObstrPulmonary` | Chronic obstructive pulmonary disease flag | Prevalence 24% |
| `ChronicCond_Depression` | Depression flag | Prevalence 36% |
| `ChronicCond_Diabetes` | Diabetes flag | Prevalence 60% |
| `ChronicCond_IschemicHeart` | Ischemic heart disease flag | Prevalence 68% |
| `ChronicCond_Osteoporasis` | Osteoporosis flag | Prevalence 27% |
| `ChronicCond_rheumatoidarthritis` | Rheumatoid arthritis flag | Prevalence 26% |
| `ChronicCond_stroke` | Stroke flag | Prevalence 8% |
| `IPAnnualReimbursementAmt` | Total **annual** inpatient reimbursement for the beneficiary | Annual total, not per claim. Median is 0 (most beneficiaries have no inpatient stay). Contains some negative values |
| `IPAnnualDeductibleAmt` | Total annual inpatient deductible paid by the beneficiary | Annual total. No nulls |
| `OPAnnualReimbursementAmt` | Total annual outpatient reimbursement | Annual total. No nulls |
| `OPAnnualDeductibleAmt` | Total annual outpatient deductible paid | Annual total. No nulls |

**Chronic condition coding.** The source has no codebook; raw values are 1 and 2 only. 1 = Yes and 2 = No was inferred for all `ChronicCond_*` columns (one coding scheme assumed across the table), then recoded to 1/0:
1. **Prevalence:** for rare conditions, code 1 is the clear minority (cancer 12%, stroke 8%). For common conditions (diabetes 60%, ischemic heart disease 68%, heart failure 49%), prevalence cannot distinguish the two readings.
2. **Cost:** code 1 has a higher P80 annual inpatient reimbursement for all 11 conditions.
3. **Consistency:** 79% of ESRD patients are coded 1 for kidney disease vs 23% of non-ESRD patients. The 21% mismatch is likely due to the fields coming from different sources (enrollment-based ESRD status vs claims-based condition flags) and anonymization; it is recorded, not corrected.

---

## 3. Inpatient claims (`Train_Inpatientdata-*.csv`)

| Column | Meaning | Null % | Notes |
|---|---|---|---|
| `BeneID` | Beneficiary ID | 0% | Foreign key → Beneficiary |
| `ClaimID` | Claim ID | 0% | Primary key; unique |
| `ClaimStartDt` | Claim start date | 0% | |
| `ClaimEndDt` | Claim end date | 0% | |
| `Provider` | Provider that submitted the claim | 0% | Foreign key → Labels |
| `InscClaimAmtReimbursed` | Amount reimbursed by the insurer for the claim | 0% | Primary cost measure (payer cost) |
| `DeductibleAmtPaid` | Deductible paid by the beneficiary for the claim | 2.2% | |
| `AttendingPhysician` | Attending physician ID | 0.3% | |
| `OperatingPhysician` | Operating physician ID | 41.1% | |
| `OtherPhysician` | Other physician ID | 88.4% | |
| `AdmissionDt` | Admission date | 0% | Inpatient only |
| `DischargeDt` | Discharge date | 0% | Inpatient only |
| `ClmAdmitDiagnosisCode` | Diagnosis code at admission | 0% | |
| `DiagnosisGroupCode` | Diagnosis-related group (DRG) code | 0% | Inpatient only. DRG version not documented in the source |
| `ClmDiagnosisCode_1` … `_10` | Diagnosis codes on the claim (1 = primary) | 0% → 90.3% | Null share rises with column number (code 2: 0.6%, code 9: 33.3%, code 10: 90.3%) |
| `ClmProcedureCode_1` | Primary procedure code | 42.8% | Read as float because of nulls (e.g. 7092.0) |
| `ClmProcedureCode_2` … `_5` | Further procedure codes | 86.5% → 99.98% | Sparse |
| `ClmProcedureCode_6` | Sixth procedure code | 100% | Entirely empty |

---

## 4. Outpatient claims (`Train_Outpatientdata-*.csv`)

Same structure as Inpatient, **except it has no** `AdmissionDt`, `DischargeDt`, or `DiagnosisGroupCode`.

| Column | Meaning | Null % | Notes |
|---|---|---|---|
| `BeneID` | Beneficiary ID | 0% | Foreign key → Beneficiary |
| `ClaimID` | Claim ID | 0% | Primary key; unique; no overlap with Inpatient |
| `ClaimStartDt` / `ClaimEndDt` | Claim start / end dates | 0% | |
| `Provider` | Provider ID | 0% | Foreign key → Labels |
| `InscClaimAmtReimbursed` | Amount reimbursed by the insurer | 0% | Primary cost measure |
| `DeductibleAmtPaid` | Deductible paid by the beneficiary | 0% | No nulls (unlike inpatient) |
| `AttendingPhysician` | Attending physician ID | 0.3% | |
| `OperatingPhysician` | Operating physician ID | 82.5% | |
| `OtherPhysician` | Other physician ID | 62.3% | |
| `ClmAdmitDiagnosisCode` | Admit / reason-for-visit diagnosis code | 79.6% | |
| `ClmDiagnosisCode_1` | Primary diagnosis code | 2.0% | 2% of outpatient claims have no primary diagnosis |
| `ClmDiagnosisCode_2` … `_10` | Further diagnosis codes | 37.7% → 99.8% | |
| `ClmProcedureCode_1` … `_6` | Procedure codes | 99.97% → 100% | Effectively empty for outpatient |

---

## Conventions used in this project

- **Cost = `InscClaimAmtReimbursed`** (insurer-paid, claim level). The annual amounts in Beneficiary are a separate aggregated view and are never added to claim-level amounts.
- **Two grains:** claim level for cost analysis; provider level for fraud analysis, because the label is per provider.
- **Race** is excluded from fraud comparisons and modelling.
- Providers are described as "flagged as potential fraud", never "fraudulent".
