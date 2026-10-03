# METSAFE: Mock Safety Surveillance Study in REDCap

A REDCap electronic data capture (EDC) project built to practise **clinical data management** and **pharmacovigilance (PV) case data capture**.

> **All data in this project is fictional.** No real patients, no real clinical data. MedDRA terms are illustrative manual entries, not licensed MedDRA coding.

**Author:** Poulami Samanta | [LinkedIn](https://linkedin.com/in/drpoulamisamanta)

---

## 1. Study concept

A mock post-marketing observational study of **metformin safety in adults with type 2 diabetes**. Participants are followed for 12 weeks, with baseline, Week 4 and Week 12 labs, and every adverse event (AE) is captured with the fields used in routine PV case processing.

| Item | Detail |
|---|---|
| Platform | REDCap v17.5.3 (demo server), classic project |
| Instruments | 9 |
| Fields | 70 |
| Participants | 12 fictional (11 enrolled, 1 screen failure) |
| Records loaded | 79 rows (12 main, 19 concomitant meds, 15 AEs, 33 lab entries) |
| Record IDs | Auto-numbered 1 to 12 |

## 2. Instruments

| # | Instrument | Repeating? | Purpose |
|---|---|---|---|
| 1 | Informed Consent | No | Consent date, version, signature (ICH-GCP) |
| 2 | Eligibility | No | Inclusion/exclusion criteria, auto-calculated eligibility |
| 3 | Demographics | No | DOB, sex, height, weight, auto-calculated age and BMI |
| 4 | Medical History | No | Diabetes duration, comorbidities, previous drug reactions |
| 5 | Study Drug Exposure | No | Formulation, dose, frequency, start/stop, status |
| 6 | Concomitant Medications | **Yes** | One entry per co-medication |
| 7 | Adverse Event | **Yes** | Core PV form (see below) |
| 8 | Lab Results | **Yes** | HbA1c, glucose, creatinine, eGFR, ALT, AST, lactate, B12 |
| 9 | End of Study | No | Completion, withdrawal or discontinuation reason |

## 3. Pharmacovigilance features (Adverse Event form)

- Verbatim AE term, plus placeholders for MedDRA Preferred Term (PT) and System Organ Class (SOC)
- Onset and stop dates, ongoing flag
- Severity (mild, moderate, severe)
- **Seriousness** (yes/no) with ICH E2A criteria: death, life-threatening, hospitalisation, disability, congenital anomaly, other medically important
- **Expectedness** against reference safety information (listed, unlisted, unknown)
- Action taken with study drug and **dechallenge** result
- Treatment given
- Outcome (ICH categories)
- **Causality (WHO-UMC):** certain, probable/likely, possible, unlikely, conditional/unclassified, unassessable
- Causality rationale, initial reporter and date report received

## 4. Design features

- **Repeating instruments** so one participant can have multiple AEs, medications and lab entries
- **Branching logic**, for example seriousness criteria appear only when Serious = Yes, and dechallenge appears only when the drug was withdrawn or interrupted
- **Calculated fields:** eligibility, age, BMI, and an ALT flag (above 3x the upper limit of normal, using a mock threshold of 120 U/L)
- **Field validation:** date formats, plausible ranges for height, weight and every lab value
- **Required fields** on the key items of each form

## 5. Data quality checks

Three custom rules were added to the built-in rules A to I.

| Rule | Logic (summary) | Expected finding |
|---|---|---|
| AE stop date before onset date | Stop and onset both filled, and stop is earlier | Record 4, instance 1 (stop 2026-05-14, onset 2026-05-17). REDCap shows 2 discrepancies because it also lists record 4's main row (no AE data), so there is 1 real error |
| Serious AE without criteria | Serious = Yes and no criterion ticked | Record 5, instance 2 (kidney function event) |
| Consent not signed | consent_signed = No | Record 11 |

The sample dataset contains these three deliberate errors, so the rules have something to find. Built-in rules for out-of-range values, wrong calculated values, invalid choices and hidden values with data all returned 0. The outlier rule flagged lab values that are clinically plausible in this population (for example lactate 9.8 mmol/L in the lactic acidosis case).

## 6. Reports

| Report | Filter | Result |
|---|---|---|
| All Adverse Events | One row per AE | 15 events |
| Serious Adverse Events | `[current-instance]<>"" and [ae_serious][current-instance]='1'` | 2 events: the true serious case (record 10, lactic acidosis) and the planted data entry error (record 5) |

## 7. Sample data highlights

- Mix of immediate and extended release metformin, three dosing frequencies, ages roughly 36 to 78
- Common AEs: nausea, diarrhoea, abdominal discomfort, metallic taste, reduced appetite
- Signal-type events: low vitamin B12, raised ALT, hypoglycaemia (with glimepiride as a confounder), rash with positive dechallenge, and one **serious lactic acidosis** case with hospitalisation and the drug withdrawn
- One unlisted AE (hair thinning) with causality rated "unlikely"
- Outcomes: 9 completed, 1 discontinued for an AE, 1 lost to follow-up, 1 screen failure

## 8. Screenshots

| | |
|---|---|
| **Instruments list** (9 forms, repeating icons) | ![Instruments list](screenshots/01_instruments_list.png) |
| **Adverse Event form** (PV fields) | ![Adverse Event form](screenshots/02_ae_form.png) |
| **Branching logic: Serious = Yes** (criteria shown) | ![Branching logic shown](screenshots/03_branching_logic.png) |
| **Branching logic: Serious = No** (criteria hidden) | ![Branching logic hidden](screenshots/03a_branching_logic_hidden.png) |
| **Record Status Dashboard** (12 participants) | ![Record status dashboard](screenshots/04_record_dashboard.png) |
| **Data quality rules and results** | ![Data quality rules](screenshots/05_data_quality_rules.png) |
| **All Adverse Events report** (15 events) | ![All AE report](screenshots/06_report_all_ae.png) |
| **Serious Adverse Events report** (2 events) | ![Serious AE report](screenshots/07_report_serious_ae.png) |
| **Repeating instruments** (participant 10) | ![Repeating instruments](screenshots/08_repeating_instruments.png) |

## 9. Repository contents

```
data_dictionary/   REDCap data dictionary (CSV)
crf/               Blank CRFs (PDF)
project_xml/       Project metadata XML (no data)
sample_data/       Fictional import file (CSV)
reports/           Exported report CSVs
screenshots/       Forms, rules, reports
```

## 10. How to reproduce

1. Create an empty REDCap project (classic data collection).
2. Upload the data dictionary under **Project Setup > Data Dictionary**.
3. Enable **Repeating instruments** and set Concomitant Medications, Adverse Event and Lab Results to repeat.
4. Import the sample data under **Data Import Tool**.
5. Add the three custom rules under **Data Quality** and build the two reports.

## 11. Limitations

- Fictional data, small sample, no real protocol
- No licensed MedDRA dictionary, so PT/SOC values are illustrative
- Classic (non-longitudinal) design: visits are captured through the Lab Results form
- No user roles, alerts or randomisation configured
- REDCap shows grey participant header rows in reports that include repeating instruments, and Data Quality may list an extra blank row for a repeating-instrument rule

## 12. Skills demonstrated

REDCap build and configuration, CRF design, data dictionary preparation, branching logic and calculated fields, data validation, data quality rules, reporting, ICH-GCP awareness, ICH E2A seriousness criteria, WHO-UMC causality assessment, adverse event case data structure.
