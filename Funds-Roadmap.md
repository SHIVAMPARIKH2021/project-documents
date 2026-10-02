# Mutual Fund & ETF Analytics Platform - Roadmap

## Vision

Build a web application that allows users to research Mutual Funds and ETFs by providing:

- Fund metadata
- Asset classification
- Benchmark information
- Holdings analysis
- AUM trends
- Risk metrics
- Exposure analytics
- Benchmark comparison

Initial focus will be on **Open-End Mutual Funds** using SEC datasets.

Later phases will add **ETF daily holdings and performance analytics**.

---

# Phase 0 - Research & Data Understanding

## SEC Datasets

### 1. N-PORT Data Sets

Source:
https://www.sec.gov/data-research/sec-markets-data/form-n-port-data-sets

Purpose:

- Holdings
- AUM
- Portfolio composition
- Derivatives
- Liquidity
- Exposure information
- Risk metrics

Primary dataset for:

- Holdings Dashboard
- Portfolio Analytics
- Exposure Analytics

---

### 2. Mutual Fund Prospectus Risk/Return (MFRR) Data Sets

Source:
https://www.sec.gov/data-research/sec-markets-data/mutual-fund-prospectus-riskreturn-summary-data-sets

Purpose:

- Investment objectives
- Strategy descriptions
- Risk disclosures
- Performance information
- Benchmark references
- Fund metadata

Primary dataset for:

- Fund Classification
- Benchmark Identification
- Active vs Passive Determination

---

# Phase 1 - Build Data Lake

## Existing Python ETL

Continue building:

```text
SEC ZIP
    ↓
Download
    ↓
Extract TSV
    ↓
Convert CSV
    ↓
Load PostgreSQL
```

### Suggested Folder Structure

```text
project-root

/data
    /nport
    /mfrr

/scripts
    download.py
    extract.py
    load.py

/database

/api

/ui
```

---

# Phase 2 - Database Design

## Fund Master

```sql
fund_master
```

Fields:

```text
fund_id
series_id
cik
fund_name
fund_family
investment_objective
benchmark_name
strategy_type
asset_class
sub_asset_class
created_date
```

---

## Reporting Period

```sql
fund_reporting
```

Fields:

```text
report_id
fund_id
report_date
net_assets
report_year
report_quarter
```

---

## Holdings

```sql
holding
```

Fields:

```text
holding_id
report_id
security_name
cusip
ticker
issuer
country
currency
shares
market_value
weight_percentage
```

---

## Bond Holdings

```sql
bond_detail
```

Fields:

```text
holding_id
coupon
maturity_date
par_value
yield_rate
```

---

## Derivatives

```sql
derivative_position
```

Fields:

```text
holding_id
contract_type
notional_amount
counterparty
```

---

## Benchmark

```sql
benchmark_master
```

Fields:

```text
benchmark_id
benchmark_name
benchmark_provider
benchmark_type
```

---

# Phase 3 - Fund Classification Engine

## Asset Class Derivation

### Equity

Rule:

```text
Equity Holdings >= 80%
```

Result:

```text
Asset Class = Equity
```

Subclasses:

```text
Large Cap
Mid Cap
Small Cap
International
Emerging Market
Sector
```

---

### Fixed Income

Rule:

```text
Bond Holdings >= 80%
```

Result:

```text
Asset Class = Fixed Income
```

Subclasses:

```text
Government Bond
Corporate Bond
Municipal Bond
High Yield
```

---

### Hybrid

Rule:

```text
Equity 30-70%
AND
Bonds 30-70%
```

Result:

```text
Hybrid
Balanced
Asset Allocation
```

---

### Alternatives

Rule:

```text
Heavy Derivatives
Commodity Exposure
Futures
Swaps
```

Result:

```text
Alternative
```

---

# Phase 4 - Active vs Passive Classification

## MFRR Strategy Analysis

Passive keywords:

```text
index
track
tracking
replicate
benchmark
index fund
passive
```

Result:

```text
PASSIVE
```

---

Active keywords:

```text
actively managed
research driven
fundamental analysis
security selection
manager discretion
```

Result:

```text
ACTIVE
```

---

# Phase 5 - Benchmark Identification

## Data Sources

Primary:

```text
MFRR Dataset
```

Target fields:

```text
Benchmark Name
Benchmark Description
Performance Comparison
```

Examples:

```text
S&P 500
Russell 1000 Growth
MSCI EAFE
MSCI World
Bloomberg Aggregate Bond
```

---

## Future Enhancement

Create:

```sql
benchmark_return
```

Fields:

```text
benchmark_id
date
index_level
daily_return
monthly_return
```

---

# Phase 6 - Java Backend

## Technology

```text
Java 21
Spring Boot
Spring Data JPA
PostgreSQL
```

---

## APIs

### Fund Search

```http
GET /api/funds
```

---

### Fund Details

```http
GET /api/funds/{id}
```

Returns:

```json
{
  "fundName": "",
  "aum": "",
  "assetClass": "",
  "benchmark": ""
}
```

---

### Holdings

```http
GET /api/funds/{id}/holdings
```

---

### Exposure

```http
GET /api/funds/{id}/exposures
```

---

### Risk Metrics

```http
GET /api/funds/{id}/risk
```

---

### Benchmark

```http
GET /api/funds/{id}/benchmark
```

---

# Phase 7 - React UI

## Technology

```text
React
TypeScript
Material UI
TanStack Table
Recharts
```

---

## Page 1 - Fund Search

Filters:

```text
Fund Name
Asset Class
Benchmark
Active/Passive
Fund Family
```

---

## Page 2 - Fund Overview

Show:

```text
Fund Name
AUM
Asset Class
Sub Asset Class
Benchmark
Strategy Type
Reporting Date
```

---

## Page 3 - Holdings

Grid:

```text
Security
Ticker
Country
Shares
Market Value
Weight
```

---

## Page 4 - Portfolio Analytics

Charts:

```text
Top Holdings
Sector Allocation
Country Allocation
Currency Allocation
```

---

## Page 5 - Risk Analytics

Show:

```text
Liquidity Buckets
Derivatives Exposure
Risk Metrics
```

---

## Page 6 - Benchmark

Show:

```text
Benchmark Name
Benchmark Provider
Fund vs Benchmark
```

---

# Phase 8 - Scheduled Updates

## Quarterly Job

Run every SEC release cycle:

```text
Download latest N-PORT
Download latest MFRR
Load Database
Refresh Aggregates
```

---

Suggested Scheduler:

```text
Spring Scheduler
OR
Airflow
OR
Cron
```

---

# Phase 9 - ETF Expansion

## Additional Data Source

ETF provider holdings files.

Examples:

```text
Vanguard
BlackRock
State Street
Invesco
```

---

## ETF Enhancements

Daily refresh:

```text
Holdings
NAV
Market Price
Premium / Discount
Tracking Error
```

---

# Phase 10 - Advanced Analytics

## Fund Screener

Filter by:

```text
Asset Class
Benchmark
AUM
Country Exposure
Risk Profile
```

---

## Portfolio Comparison

Compare:

```text
Fund A
vs
Fund B
```

Show:

```text
Overlapping Holdings
Country Exposure
Sector Exposure
Benchmark
```

---

## Benchmark Analytics

Show:

```text
Fund Return
Benchmark Return
Alpha
Beta
Tracking Error
```

---

# MVP Deliverable (First Release)

The first public version should deliver:

✅ Fund Search

✅ Fund Overview

✅ Holdings

✅ AUM

✅ Asset Class Classification

✅ Active/Passive Classification

✅ Benchmark Identification

✅ Country Exposure

✅ Currency Exposure

✅ Liquidity Classification

✅ Risk Metrics

✅ Quarterly Automated SEC Data Ingestion

Target timeline:

```text
Month 1
---------
ETL + Database

Month 2
---------
Spring Boot APIs

Month 3
---------
React Dashboard + Classification Engine

Month 4
---------
Benchmark Module + Production Deployment
```
---

# Milestone-1: Core Ingestion, Fund Universe Taxonomy & Classification

## Current Status & Roadmap Alignment
- **Where We Are:** Transitioning from Phase 1 (Data Lake & Ingestion) to Phases 2–5 (Database Design, Asset Classification, Active/Passive Rules, and Benchmark Normalization).
- **Milestone-1 Objective:** Ingest SEC EDGAR disclosure feeds, accurately segment fund structures across open-end funds, closed-end funds, ETFs, and UITs, extract investment strategies, and determine active vs. passive execution along with portfolio turnover dynamics.

---

## Fund Universe Taxonomy, Turnover Dynamics & SEC/EDGAR Identification

| Fund Type | Portfolio Changing / Turnover Frequency | Regulatory Form / Filing Type | Primary EDGAR / SEC Dataset & Schema Identifiers | Identification & Disambiguation Rules |
| :--- | :--- | :--- | :--- | :--- |
| **Open-End Mutual Funds (Mutual Funds / OEFs)** | **Semi-Annual / Quarterly disclosures** *(Internal daily rebalancing via cash flow & redemption/creation, but public portfolio disclosures are submitted quarterly)* | **Form N-PORT** (Quarterly portfolio holdings), **Form N-CEN** (Annual census), **Form 485BPOS / 497** (Prospectus) | • **MFRR Data Sets** (`sec_financial` / `sub.tsv`, `txt.tsv`, `num.tsv`)<br>• **N-PORT Datasets** (`sec_financial.nport_holdings`)<br>• **Company Tickers MF** (`company_tickers_mf.json`) | • `sub.tsv`: `form` IN (`485BPOS`, `497`, `N-1A`)<br>• `company_tickers_mf.json`: Maps CIK $\rightarrow$ Series ID (`seriesId`) $\rightarrow$ Class ID (`classId`) $\rightarrow$ Ticker symbol.<br>• `N-CEN` / `sub.tsv`: `inv_company_type` = `N-1A` and `is_etf` = `N` / `false`. |
| **Exchange-Traded Funds (ETFs)** | **Daily basket disclosures / creation-redemption rebalancing** *(Public SEC N-PORT filed quarterly with 60-day lag; Authorized Participant baskets updated daily)* | **Form N-PORT**, **Form N-CEN**, **Form 485BPOS** (Unit Investment Trust ETFs file Form N-8B-2 / S-6 or N-CEN) | • **N-PORT Data Sets**<br>• **MFRR Data Sets**<br>• **SEC N-CEN Data** (`sec_financial.ncen`) | • `N-CEN`: Flag `is_etf = 'Y'` / `true`.<br>• `sub.tsv` / Prospectus: Regulated under `N-1A` (open-end fund) or `N-4` / `N-8B-2` with `ETF` designated in `series_name` or `StrategyNarrativeTextBlock`.<br>• SEC Series-Level Tag: `FormType` / `ClassContractType` indicates exchange-traded class shares. |
| **Closed-End Funds (CEFs)** | **Quarterly disclosures** *(Low to moderate turnover; fixed share capital, no daily share creation/redemption)* | **Form N-PORT**, **Form N-CSR / N-CSRS**, **Form N-2** (Registration Statement) | • **SEC N-PORT Datasets**<br>• **EDGAR Company Submissions / N-2 filings** | • Regulated under Form `N-2` (`form` = `N-2`, `N-2/A`).<br>• `N-CEN`: `inv_company_type` = `N-2`.<br>• Typically trades under a single ticker on major exchanges without multiple share classes (`seriesId`/`classId` mapping absent in `company_tickers_mf.json`). |
| **Unit Investment Trusts (UITs)** | **Fixed / Static (Yearly or buy-and-hold for fund lifespan)** *(No active manager trading; securities held until fixed maturity date)* | **Form N-CEN**, **Form S-6**, **Form N-8B-2** | • **SEC N-CEN Datasets**<br>• **EDGAR Registration Data** | • `N-CEN`: `inv_company_type` = `N-4` or `UIT`.<br>• `form` IN (`S-6`, `N-8B-2`).<br>• Portfolio turnover rate reported near 0% in prospectus/census feeds. |
| **Money Market Funds (MMFs)** | **Daily / Ultra-Short** *(Continuous daily maturities, 60-day maximum weighted average maturity (WAM))* | **Form N-MFP / N-MFP2** (Monthly detailed portfolio schedule), **Form N-CR** (Credit events) | • **SEC Form N-MFP Data Sets** | • `form` IN (`N-MFP`, `N-MFP2`).<br>• MFRR / N-PORT: Category tag or strategy text contains `Money Market Fund` (Rule 2a-7 under Investment Company Act of 1940). |

---

## EDGAR Schema Extraction & Table Mapping (`sec_financial`)

---

# Milestone-1: Core Ingestion, Fund Universe Taxonomy & Schema Classification

## 1. Where We Are
- **Completed Infrastructure:** Raw staging schema (`sec_financials`), analytics datastore (`analytics`), compliance rules (`analytics.compliance_rules`), pipeline execution logs, and Spring Batch metadata (`batch.*`) are fully provisioned via Flyway migrations (V1–V10).
- **Current Objective:** Wire the Spring Batch transformation engine from raw staging tables (`sec_financials.submissions`, `text_disclosures`, `numeric_facts`, `company_tickers_mf`) into curated analytics tables (`analytics.fund_master`, `fund_classes`, `benchmark_master`).
- **Core Milestone-1 Deliverable:** Robustly identify and classify the distinct fund types, map tickers to fund series, extract strategy narratives, and apply compliance rules to establish active vs. passive designations.

---

## 2. Fund Universe Taxonomy & Portfolio Rebalancing Dynamics

| Fund Structure | Portfolio Turnover & Rebalancing Frequency | SEC Filing Types | Primary SEC Staging Source |
| :--- | :--- | :--- | :--- |
| **Open-End Mutual Funds (OEFs)** | **Semi-Annual / Quarterly public reporting** *(Continuous internal trading to satisfy shareholder inflows and redemptions; public filings lag by 60 days).* | `485BPOS`, `497`, `N-1A`, `N-PORT` | `sec_financials.submissions`, `text_disclosures`, `company_tickers_mf` |
| **Exchange-Traded Funds (ETFs)** | **Daily basket disclosures / continuous creation-redemption** *(Daily AP basket composition published every morning; full portfolio disclosures filed quarterly on Form N-PORT).* | `485BPOS`, `N-1A`, `N-PORT`, `N-CEN` | `sec_financials.submissions`, `text_disclosures`, `company_tickers_mf` |
| **Closed-End Funds (CEFs)** | **Quarterly / Semi-Annual public reporting** *(Static capital base; trades on secondary markets like an equity; portfolio turnover driven entirely by manager discretion).* | `N-2`, `N-2/A`, `N-CSR`, `N-PORT` | `sec_financials.submissions`, `text_disclosures` |
| **Unit Investment Trusts (UITs)** | **Static / Buy-and-Hold for fund lifespan** *(Fixed portfolio defined at inception; no active ongoing security replacement).* | `S-6`, `N-8B-2`, `N-CEN` | `sec_financials.submissions` |
| **Money Market Funds (MMFs)** | **Daily / Ultra-Short** *(High-frequency roll-over of short-term instruments; subject to 60-day maximum Weighted Average Maturity).* | `N-MFP`, `N-MFP2`, `N-CR` | `sec_financials.submissions`, `numeric_facts` |

---

## 3. Disambiguating Fund Types in `sec_financials`

### A. Open-End Mutual Funds (OEFs)
* **Registration Form:** `sec_financials.submissions.form IN ('485BPOS', '497', 'N-1A')`
* **Ticker & Share Class Mapping:** Filter `sec_financials.company_tickers_mf` where `series_id` matches the fund series and `class_id` represents individual share classes (e.g., Class A, Institutional).
* **Strategy Narrative:** Disclosed under tag `StrategyNarrativeTextBlock` in `sec_financials.text_disclosures`.
* **Identification Logic:**
  ```sql
  SELECT 
      s.cik,
      s.adsh,
      s.name AS registrant_name,
      s.form,
      m.series_id,
      m.class_id,
      m.symbol AS class_ticker
  FROM sec_financials.submissions s
  JOIN sec_financials.company_tickers_mf m ON s.cik = m.cik
  WHERE s.form IN ('485BPOS', '497')
    AND m.symbol NOT ILIKE '%ETF%';
    ```
### B. Exchange-Traded Funds (ETFs)
* **Registration Form:** Filed under `485BPOS` or `N-1A` (similar to mutual funds).
* **Identification Rules:**
  * Registrant name (`s.name`) or class title contains terms like `' ETF'`, `'Exchange-Traded Fund'`, or `'Trust'`.
  * MFRR strategy text (`txt.tsv`) contains explicit ETF creation/redemption disclosures (`Authorized Participant`, `creation units`, `secondary market`).
  * `company_tickers_mf.symbol` typically reflects single/exchange-traded share classes rather than traditional mutual fund class structures.

### C. Closed-End Funds (CEFs)
* **Registration Form:** `sec_financials.submissions.form IN ('N-2', 'N-2/A')`.
* **Identification Rules:**
  * CEFs are absent from `company_tickers_mf` because they do not have separate multi-class mutual fund series structures.
  * Identification relies on `sub.form LIKE 'N-2%'` and ticker resolution via standard equity ticker directories.

### D. Unit Investment Trusts (UITs)
* **Registration Form:** `sec_financials.submissions.form IN ('S-6', 'N-8B-2')`.
* **Identification Rules:**
  * `form` values identify the trust structure directly.
  * Zero or near-zero turnover disclosures in strategy notes.

---

## 4. Milestone-1 Execution Flow

1. **Ingest & Stage:** Verify raw feeds populate `sec_financials.submissions`, `text_disclosures`, `numeric_facts`, and `company_tickers_mf`.
2. **Filter Open-End Funds & ETFs:** Select submissions with `form IN ('485BPOS', '497')`.
3. **Parse Strategy & Classify:**
   * Join `sec_financials.text_disclosures` on `tag = 'StrategyNarrativeTextBlock'`.
   * Apply rules from `analytics.compliance_rules` ordered by `priority ASC`:
     * Evaluate Priority 10–13 (Exclusions/Negations like `does not seek to replicate`).
     * Evaluate Priority 20–29 (Passive patterns like `tracks the performance of`).
     * Evaluate Priority 30–39 (Active patterns like `actively managed`, `fundamental research`).
4. **Populate Master Tables:**
   * Upsert unique `series_id` records into `analytics.fund_master` with determined `strategy_type` (`ACTIVE`, `PASSIVE`, `UNKNOWN`).
   * Populate associated share classes and tickers into `analytics.fund_classes`.

### 1. ```fundsImportJob``` Basic understanding

1. Are We Only Importing Funds That Change Yearly?
    - Yes, for this specific job (```fundsImportJob```).
Because your reader filters by td.tag = 'StrategyNarrativeTextBlock', you are reading Statutory Prospectuses (Form N-1A / 485BPOS). Under SEC Rule 485, open-end funds (mutual funds and open-end ETFs) must update their prospectus annually within 120 days of their fiscal year-end.
The data being loaded into fund_master represents Static Master & Policy Data:
Legal identity (fund_name, fund_family, series_id, cik)
Mandates (investment_objective, strategy_narrative, strategy_type)
Regulatory primary benchmarks (benchmark_id, benchmark_name)
These attributes rarely change mid-year unless the fund files a material prospectus supplement (Form 497 / "sticker").

2. Which Type of Funds Are These?
    - Because your query joins against ```sec_financials.company_tickers_mf on series_id (format S0000xxxxx)```:
**Open-End Mutual Funds:** Traditional retail/institutional share-class mutual funds (e.g., Vanguard 500 Index Admiral, Fidelity Contrafund).
**Open-End ETFs (Exchange-Traded Funds):** ETFs registered under the Investment Company Act of 1940 on Form N-1A (e.g., SPDR SPY is a UIT, but Vanguard VOO or iShares IVV file on Form N-1A).
These are not Closed-End Funds (CEFs file Form N-2) or Money Market Funds reporting daily shadow NAVs (Form N-MFP).