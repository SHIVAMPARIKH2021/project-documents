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
(Check the section - What Information Can We Extract and Show on the UI)
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


Here is the architectural plan and roadmap section to append to **`Funds-Roadmap.md`**.

---

## Page-2: What Information Can We Extract and Show on the UI?

When `class IS NULL AND measure IS NULL` in `sec_financials.numeric_facts`, the SEC is giving you **fund-level core truth** that is unpolluted by share-class fee variations:

1. **Fund-Level Fee Baseline:**
* **Management Fee Rate (`ManagementFeesOverAssets`)**: Shows the base advisory fee the portfolio manager charges, irrespective of 12b-1 retail distribution cuts.
* **Gross Expense Ratio (`ExpensesOverAssets`)**: Directly populates the baseline operating cost for single-class funds or ETFs.


2. **Historical Performance Volatility (Quarterly Extremes):**
* **Best Quarter (`BarChartHighestQuarterlyReturn`)**: The highest calendar-quarter return (e.g., `+23.4% in Q2 2020`).
* **Worst Quarter (`BarChartLowestQuarterlyReturn`)**: The lowest calendar-quarter return (e.g., `-18.2% in Q1 2020`).
* These provide **instant downside/upside risk context** without having to calculate daily prices from scratch.


3. **Calendar Year Historical Return Trajectory (`AnnlRtrPct`):**
* Produces the official 10-year historical annual returns bar chart mandated by SEC Item 4(b)(2).


4. **Projected Hypothetical Cost Simulation (`ExpenseExampleYear01` through `Year10`):**
* Shows the standardized SEC cost to hold the fund on a $10,000 investment over 1, 3, 5, and 10 years.



---

### UI Page Addition: Fund Overview & Fee Breakdown Tabs

On **Page 2 (Fund Overview)** or a new dedicated **"Fees & Performance History"** tab:

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Vanguard 500 Index Fund (Series: S000006037)                            │
│  [Passive / Index]  •  Benchmark: S&P 500 Index  •  Cadence: Annual      │
├──────────────────────────────────────────────────────────────────────────┤
│  FUND-LEVEL EXPENSE BASELINE                                             │
│  Management Fee: 0.03%  |  Net Baseline Expense: 0.04%                   │
│  Cost on $10k (1-Yr): $4 | (3-Yr): $13 | (5-Yr): $23 | (10-Yr): $51      │
├──────────────────────────────────────────────────────────────────────────┤
│  HISTORICAL EXTREMES & VOLATILITY RANGE (Form N-1A Bar Chart)            │
│  ▲ Best Quarter:   +20.54% (Q2 2020)                                     │
│  ▼ Worst Quarter:  -19.60% (Q1 2020)                                     │
│  Historical Trajectory: [ 2016: 11.9% | 2017: 21.8% | ... | 2024: 25.0% ]│
└──────────────────────────────────────────────────────────────────────────┘

```

---

### Markdown to Append to `Funds-Roadmap.md`

Append this block directly to the end of your `Funds-Roadmap.md` file:

```markdown
---

# Milestone-2: Fund-Level Baseline Metrics & Performance Extremes Engine

## 1. Context & Business Value
Rows in `sec_financials.numeric_facts` where both `class IS NULL` and `measure IS NULL` capture **Series-Level Default Disclosures** without share-class dimensional noise. Ingesting these provides essential master-level fee baselines, expense projections, and historical volatility extremes (best/worst quarters and calendar-year bar chart returns) required under SEC Form N-1A Item 4(b)(2).

---

## 2. Target Schema Additions

### Table Enhancement: `analytics.fund_master`
```sql
ALTER TABLE analytics.fund_master
    ADD COLUMN IF NOT EXISTS management_fee_pct       NUMERIC(6, 4),
    ADD COLUMN IF NOT EXISTS net_expense_ratio        NUMERIC(6, 4),
    ADD COLUMN IF NOT EXISTS best_quarter_return      NUMERIC(6, 4),
    ADD COLUMN IF NOT EXISTS best_quarter_period      VARCHAR(10),
    ADD COLUMN IF NOT EXISTS worst_quarter_return     NUMERIC(6, 4),
    ADD COLUMN IF NOT EXISTS worst_quarter_period     VARCHAR(10);

```

### New Table: `analytics.fund_annual_returns` (Bar Chart History)

```sql
CREATE TABLE IF NOT EXISTS analytics.fund_annual_returns (
    id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    series_id         VARCHAR(20) NOT NULL REFERENCES analytics.fund_master(series_id) ON DELETE CASCADE,
    return_year       INT NOT NULL,
    return_pct        NUMERIC(7, 4) NOT NULL,
    accession_number  VARCHAR(25),
    created_at        TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT uq_fund_year_return UNIQUE (series_id, return_year)
);

CREATE INDEX IF NOT EXISTS idx_fund_annual_returns_series 
ON analytics.fund_annual_returns(series_id);

```

---

## 3. Component Architecture & Build Roadmap

### A. Batch Job Components (`fundMetricsJob` or Step in `fundsImportJob`)

* **Reader (`FundMasterMetricsReader`):**
* Extract facts from `sec_financials.numeric_facts` where `class IS NULL AND measure IS NULL`:
* Target Tags:
* Fees: `ManagementFeesOverAssets`, `ExpensesOverAssets`, `OtherExpensesOverAssets`
* Extremes: `BarChartHighestQuarterlyReturn`, `BarChartLowestQuarterlyReturn`
* Returns: `AnnlRtrPct`
* Expense Examples: `ExpenseExampleYear01`, `Year03`, `Year05`, `Year10`




* **Processor (`FundMasterMetricsProcessor`):**
* Aggregate metrics per `series_id`.
* Parse quarterly dates associated with high/low return facts (from context period attributes).
* Prepare `fund_master` patch payload and `List<FundAnnualReturn>` payload.


* **Writer (`FundMasterMetricsWriter`):**
* Bulk upsert into `analytics.fund_master` to update fee/extreme columns.
* Bulk upsert into `analytics.fund_annual_returns` using `ON CONFLICT (series_id, return_year) DO UPDATE`.



---

### B. Java Backend API Layer

* **Endpoint 1: Fund Profile & Fee Baseline**
```http
GET /api/funds/{seriesId}/profile

```


*Response:* Master metadata, strategy narrative, objective, `managementFeePct`, `netExpenseRatio`, and benchmark association.
* **Endpoint 2: Performance Extremes & Annual Returns**
```http
GET /api/funds/{seriesId}/performance-history

```


*Response:*
```json
{
  "seriesId": "S000006037",
  "bestQuarter": { "returnPct": 0.2054, "period": "2020-Q2" },
  "worstQuarter": { "returnPct": -0.1960, "period": "2020-Q1" },
  "calendarYearReturns": [
    { "year": 2021, "returnPct": 0.2871 },
    { "year": 2022, "returnPct": -0.1811 },
    { "year": 2023, "returnPct": 0.2629 },
    { "year": 2024, "returnPct": 0.2502 }
  ]
}

```



---

### C. Frontend / UI Components

* **Component 1: Fund Header Quick-Stats Badge**
* Displays Management Fee vs. Benchmark comparison badge.
* Displays Upside/Downside Quarterly Extreme spread (`▲ Best: +20.5% | ▼ Worst: -19.6%`).


* **Component 2: Annual Performance Bar Chart (`Recharts` / `TanStack`)**
* Interactive bar chart rendering `calendarYearReturns` across the fund’s 10-year lifespan.


* **Component 3: Cost-Over-Time Calculator Widget**
* Computes hypothetical holding cost based on `ExpenseExampleYear01` through `Year10` for customizable investment balances ($1,000 to $100,000).



```

```
