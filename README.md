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



# Socio-Economic Baseline & Cross-Domain Analysis Report
Assessment Period: 30-Day Synthesis

Scope: Multi-Village Community Study (
N=120 Beneficiaries)

Author: Data Analyst / Scientist

## 1. Executive Summary
This report delivers a comprehensive exploratory analysis of socio-economic, health, and financial indicators across 120 community beneficiaries spanning six distinct villages (Gatwekera, Laini Saba, Lindi, Mashimoni, Silanga, and Soweto East). By merging fragmented operational datasets into a unified Master Analytical Dataset (MAD), this study uncovers critical intersections between household debt, health clinic utilization, and community program engagement.

## Key Highlights:
High Financial Participation: 71.7% of all beneficiaries maintain active loan portfolios through local SACCO structures.

Geographic Disparities: Credit intensity is heavily localized, with Laini Saba acting as the primary financial hub, capturing 24.69% of total borrowed capital despite housing 19.47% of the population.

The Health-Debt Nexus: Statistical correlation analysis reveals a positive link (
r=0.272) between outstanding loan balances and health clinic visits, indicating that healthcare needs and financial stress frequently compound at the household level.

## 2. Introduction & Background
Community development initiatives often suffer from data fragmentation, where health clinics, microfinance programs, and social activities operate in silos. This 30-day assessment was initiated to establish a unified view of community welfare. By bridging cross-domain data sources, this report aims to provide local leaders and stakeholders with empirical evidence to guide resource allocation, target vulnerable cohorts, and optimize program delivery.

## 3. Methodology & Data Pipeline
To overcome data siloing and prevent row-duplication errors ("row explosion") common in multi-table relational data, the analysis followed a rigorous data engineering pipeline:

1. Relational Ingestion & Cleaning: Standardized and cleaned five core operational tables: Beneficiaries, Health Visits, Program Activities, SACCO Loans, and Budgets.

2. Aggregated Feature Engineering: Using Pandas groupby with named aggregation (.agg), individual event logs (e.g., multiple clinic visits or separate loan records) were compressed down to exact individual-level summary metrics per beneficiary (TotalHealthVisits, TotalLoans, TotalBorrowed_KES, TotalOutstanding_KES, TotalActivities).

3. Master Analytical Dataset (MAD) Construction: Performed left-joins anchored on BeneficiaryID against the base demographic table, resulting in a clean, non-duplicated 
(120,15) Master Analytical Dataset. Missing activity and financial metrics for non-participants were systematically zero-filled to preserve statistical integrity.

## 4. Exploratory & Geographic Analysis
Analysis of the Master Analytical Dataset reveals pronounced spatial variations across the six target communities (
N=120).

* Village Population and Credit Distribution Breakdown:
Mashimoni: Houses the largest population cohort with 23 beneficiaries (20.35%), accounting for 740,000 KES in total borrowing (16.46%).

* Laini Saba: Comprising 22 beneficiaries (19.47%), Laini Saba leads all communities in credit scale, driving 1,110,000 KES in total borrowing (24.69% of the aggregate capital pool) alongside high program activity participation.

* Lindi: Matches Laini Saba with 22 beneficiaries (19.47%), maintaining significant borrowing of 905,000 KES (20.13%) alongside elevated healthcare demands (averaging 1.82 health visits per resident).

* Gatwekera: Accounts for 20 beneficiaries (17.70%) with 835,000 KES in borrowing (18.58%).

* Soweto East: Represents 19 beneficiaries (16.81%) with 640,000 KES in borrowing (14.24%), while recording the highest average health visits per resident (1.84).

* Silanga: A micro-community of 7 beneficiaries (6.19%), registering proportional totals with 265,000 KES in borrowing (5.90%).

## 5. Statistical Validation & Cross-Domain Insights
To evaluate structural relationships across domains, a correlation matrix and corresponding heatmap were analyzed across numerical features:

* The Credit-Health Nexus (
r=0.272): A positive correlation exists between Total Outstanding KES (and total loans) and Total Health Visits. This quantitative link validates that households managing heavier debt burdens also experience or require higher health service utilization, signaling compounding vulnerabilities.

* Internal Financial Leverage (
r=0.831): A strong positive correlation between total borrowed funds and outstanding balances indicates persistent credit exposure rather than rapid revolving turnover across the population.

* Program Independence (
r≈−0.01 to 
0.14): Community activity participation operates independently of immediate financial debt and age metrics, suggesting that social program engagement reaches participants across diverse financial strata without direct friction from credit obligations.

## 6. Strategic Recommendations & Action Plan
Based on empirical findings from the 30-day assessment, the following actions are recommended for stakeholders:

1. Targeted Health & Financial Relief in Lindi & Soweto East: Given the convergence of high clinic utilization and substantial debt, introduce integrated financial literacy programs alongside health subsidies to prevent medical debt spirals.

2. Credit Risk Management in Laini Saba: Since Laini Saba anchors nearly a quarter of all borrowed capital, establish localized advisory services to support healthy portfolio turnover and repayment tracking.

3. Equitable Program Expansion: Leverage the independence of community activities (
r≈0) to utilize social programs as engagement channels for vulnerable populations in Silanga and Mashimoni who may be isolated from formal credit or health networks.
