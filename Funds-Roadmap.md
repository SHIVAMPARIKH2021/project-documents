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