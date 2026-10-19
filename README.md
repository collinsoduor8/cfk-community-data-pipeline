# cfk-community-data-pipeline
End-to-end M&amp;E data pipeline and analysis covering beneficiary registries, health outreach, program activities, SACCO loans, and project budgets. Built with Python &amp; Pandas for automated data cleaning, orphan ID auditing, and operational insights.                   
# CFK Community Data Pipeline & Audit Engine

An automated end-to-end Python and Pandas data cleaning, standardization, and auditing pipeline designed for community-level M&E (Monitoring & Evaluation) datasets. This project processes multi-sectoral operational data covering beneficiary registries, health clinic visits, youth/sports program activities, SACCO micro-finance loans, and project budgets.

---

## Project Overview & Objectives

Community-level programs often ingest data from various field officers, leading to messy records, inconsistent text casing, mixed date formats, and broken cross-table relationships. This pipeline automates the data hygiene process by:
* Standardizing Text & Categorical Variables: Cleaning messy text strings, HTML entities (`&nbsp;`), and mapping fragmented categories (e.g., gender variants like `m`, `F`, `FEMALE` unified into standard titles).
* Currency & Numerical Parsing: Stripping currency symbols (`KES`), commas, and handling missing numerical strings safely.
* Date Parsing: Unifying mixed-format date strings into standard ISO datetime objects (`YYYY-MM-DD`).
* Cross-Table Integrity Auditing: Implementing automated record-matching checks (e.g., identifying unregistered walk-in visits or orphan IDs across operational files).

---

## Datasets Processed

The pipeline ingests and cleans 5 core operational datasets:
1. Beneficiaries (`CFK2_Beneficiaries.csv`): Master demographic registry tracking 120 individuals across Kibera villages (Gatwekera, Soweto East, Laini Saba, Mashimoni).
2. Health Visits (`CFK2_HealthVisits.csv`): 202 clinical logs detailing diagnoses, consultations, medication costs, and insurance coverage.
3. Program Activities (`CFK2_ProgramActivities.csv`): 180 records tracking attendance, grades, and session durations across sports, education, and empowerment programs.
4. SACCO Loans (`CFK2_SACCOLoans.csv`): 150 micro-finance loan applications, disbursement dates, repayment tracking, and default statuses.
5. Project Budgets (`CFK2_ProjectBudgets.csv`): 100 financial lines detailing donor allocations, actual spends, utilization percentages, and audit statuses.

---

## Key Pipeline Features & Code Highlights

### 1. Robust Modular Helper Functions

Instead of repetitive code, custom helper functions handle standard cleaning tasks globally:
```python
def clean_text(df, columns):
    for col in columns:
        if col in df.columns:
            df[col] = df[col].str.replace('&nbsp;', ' ', regex=True).str.strip().str.title()
    return df

def clean_currency_col(val):
    if pd.isna(val):
        return np.nan
    if isinstance(val, str):
        val = val.replace('KES', '').replace(',', '').strip()
    try:
        return float(val)
    except:
        return np.nan


for name, df in operational_tables.items():
    df['RecordStatus'] = df['BeneficiaryID'].apply(
        lambda x: 'Registered Member' if x in registered_ids else 'Unregistered / Walk-in'
    )

----

### 2. Standardizing Categorical Mappings

Categorical inconsistencies are programmatically mapped to clean, uniform standards:
* **Gender Normalization:** Unified variations (`m`, `f`, `FEMALE`, etc.) into clean standard titles.
* **Boolean Conversion:** Mapped messy string/numeric boolean flags (`YES`, `Y`, `1`, `True`, `0`) cleanly into true Python booleans (`True`/`False`).

### 3. Automated Cross-Table Auditing (`RecordStatus` Flag)

To catch data anomalies where operational activities or health visits reference IDs not present in the master beneficiary registry, the pipeline dynamically flags records:

```python
registered_ids = set(beneficiaries['BeneficiaryID'])
operational_tables = {
    'Health Visits': health_visits,
    'Program Activities': activities,
    'SACCO Loans': loans
}

for name, df in operational_tables.items():
    df['RecordStatus'] = df['BeneficiaryID'].apply(
        lambda x: 'Registered Member' if x in registered_ids else 'Unregistered / Walk-in'
    )

