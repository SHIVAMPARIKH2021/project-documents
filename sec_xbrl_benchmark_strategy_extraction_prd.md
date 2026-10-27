# Product Requirements Document (PRD)

## SEC XBRL Fund Benchmark Ingestion, Reporting Cadence & Strategy Engine

- **Document Version:** 2.0.0
- **Target Pipeline:** Spring Batch SEC Ingestion Pipeline (`funds-import-batch`)
- **Target Schemas:** `sec_financials` (Source), `analytics` (Serving / Master Warehouse)
- **Status:** APPROVED ARCHITECTURE SPECIFICATION

---

## 1. Executive Summary & Business Objective

The analytics data pipeline extracts mutual fund master records, performance benchmarks, and investment strategies from SEC XBRL Risk/Return (RR) filings (Forms 485APOS, 485BPOS, and 497).

Currently, funds default to `benchmark_id = 0` (`"Not Disclosed / New Inception"`) or have their legitimate benchmarks overwritten during quarterly runs due to:
1. Rigid tag filtering that omits valid comparative return tags (such as `AvgAnnlRtrPct`).
2. Dimensional inversion where filers store benchmark members in `class` rather than `measure`.
3. Lack of filing-level point-in-time benchmark cadence tracking: when funds submit mid-year fee supplements or omit annual returns in subsequent quarters, their operational reporting status is not updated deterministically.

The objectives of this specification are:
1. Normalize and resolve 100% of legally disclosed SEC benchmarks natively during ingestion.
2. Track filing-level benchmark reporting cadence dynamically across quarters in Java (`FundsItemProcessor`), toggling cadence flags based on quarterly disclosure presence without losing historical association links.
3. Classify fund investment strategies (`ACTIVE`, `PASSIVE`, `UNKNOWN`) via the prioritized `analytics.compliance_rules` engine.
4. Maintain a clean separation of concerns: perform business rule state evaluation in Java and direct CRUD/upsert persistence in SQL without changing the 3-column DDL of `analytics.staged_filing_benchmark`.

---

## 2. Root Cause Analysis

### 2.1 Missing Performance Return Tags
- **Issue:** Extraction queries strictly filter on `AverageAnnualReturnYear01/05/10`.
- **Reality:** Numerous mutual fund series disclose primary benchmark returns under alternate taxonomy tags, notably `AvgAnnlRtrPct`.

### 2.2 Dimensional Inversion (`class` vs. `measure`)
- **Issue:** Ingestion engines assume index tokens live exclusively in `numeric_facts.measure` and that `numeric_facts.class` only holds share classes (`C000...`).
- **Reality:** Filers frequently place custom benchmark members (e.g., `StandardPoors500Index`) in `nf.class` while leaving `measure` null. Filtering out populated `class` values drops valid benchmark rows.

### 2.3 Presentation Label Member Suffix Disconnect
- **Issue:** Strict equality joins between raw fact tokens and label tags (`nf.measure = pl.tag`) fail.
- **Reality:** `sec_financials.numeric_facts` records the raw token without standard XBRL suffixes, while `sec_financials.presentation_labels` tags end with `Member` (and often prepend `index_` or `rr_`).

### 2.4 Cadence Drift & Point-in-Time Status Loss
- **Issue:** Mutual fund companies report quantitative returns annually (Forms 485BPOS/485APOS) but submit non-return fee and administrative filings (Form 497) throughout intermediate quarters.
- **Reality:** If a fund reports an index in Q1-2023 but submits a filing without return disclosures in Q2-2023, the fund does not have an active benchmark disclosure for Q2-2023. Overwriting `fund_master` with static values across all quarters without updating cadence flags obscures whether a fund currently maintains an active quarterly return cadence.

### 2.5 Tax Metric Contamination
- **Issue:** Return tables include tax-adjusted calculations (`AfterTaxesOnDistributions`, `AfterTaxesOnDistributionsAndSales`) on fund share classes.
- **Reality:** If not explicitly excluded, tax metrics are erroneously registered in `benchmark_master` as broad-market index entities.

---

## 3. End-to-End System Architecture

```
                       [ SEC XBRL Submissions ]
             (numeric_facts, text_disclosures, presentation_labels)
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│               STEP 1: Benchmark Extraction & Staging                   │
│                                                                        │
│  BenchmarkItemReader:                                                  │
│    - Scan numeric_facts across tags:                                   │
│      AverageAnnualReturnYear01/03/05/10/15/25, AvgAnnlRtrPct           │
│    - Extract token from measure; fallback to non-C000... class         │
│    - Exclude AfterTaxes* metrics                                       │
│                                                                        │
│  BenchmarkItemProcessor:                                               │
│    - Resolve human-readable name via presentation_labels (Member joins)│
│    - Strip boilerplate and match Provider Rules (S&P, Russell, etc.)   │
│                                                                        │
│  BenchmarkItemWriter:                                                  │
│    - Upsert analytics.benchmark_master                                 │
│    - Insert analytics.staged_filing_benchmark (accession, id, created) │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│            STEP 2: Fund Processing & Cadence Resolution                │
│                                                                        │
│  FundsItemProcessor (State & Cadence Logic in Java):                   │
│    - Read current filing benchmarks from staged_filing_benchmark cache │
│    - Classify Strategy: Match narrative against compliance_rules       │
│    - Evaluate Quarterly Cadence & Benchmark Status:                    │
│        * Disclosed in current filing:                                  │
│            benchmark_id = resolved_id,                                 │
│            reporting_cadence = 'ANNUAL', has_daily_pricing = TRUE      │
│        * Not disclosed in current filing:                              │
│            benchmark_id = 0, benchmark_name = 'Not Disclosed...',      │
│            reporting_cadence = 'SUPPLEMENTAL',                         │
│            has_daily_pricing = FALSE (cadence flag set to false)       │
│    - Map primary benchmark association flag (is_primary = TRUE)        │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                     STEP 3: Relational Persistence                     │
│                                                                        │
│  FundsItemWriter:                                                      │
│    - analytics.fund_master (Active operational profile)                │
│    - analytics.fund_benchmark_association (Audit per accession/series)│
│    - analytics.quarter_fund_association (Historical quarterly cohort)  │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Functional Requirements (FR)

The Functional Requirements are structured as modular specifications (`FR-xxx`) to allow frictionless additions of future regulatory rules, taxonomy tags, and cadence behaviors.

### [FR-100] Benchmark Ingestion & Taxonomy Normalization

#### FR-101: Unified Return Tag Coverage
The extraction reader must query both standard Form N-1A tags and alternative taxonomy tags.
- Required tag list:
  - `AverageAnnualReturnYear01`, `AverageAnnualReturnYear03`, `AverageAnnualReturnYear05`
  - `AverageAnnualReturnYear10`, `AverageAnnualReturnYear15`, `AverageAnnualReturnYear25`
  - `AverageAnnualReturnSinceInception`, `AverageAnnualReturnSinceInception1`, `AverageAnnualReturnSinceInception2`
  - `AnnualReturnYear01`, `AnnualReturnYear05`, `AnnualReturnYear10`
  - `AvgAnnlRtrPct`

#### FR-102: Dimensional Token Resolution Hierarchy
Candidate benchmark tokens must be extracted dynamically from `sec_financials.numeric_facts`:
1. **Primary Candidate:** `nf.measure` if non-null, non-empty, and not matching internal tax exclusions (`AfterTaxes*`, `ReturnBeforeTaxes*`).
2. **Secondary Candidate:** If `nf.measure` is blank/null, evaluate `nf.class`. If `nf.class` is non-null, non-empty, and does **not** match share-class pattern `^C[0-9]{9}`, extract `nf.class` as the benchmark token.
3. If both fields are blank or represent fund share classes / tax adjustments, discard the fact row.

#### FR-103: Presentation Label Taxonomy Linkbase Matching
To resolve the human-readable display title in `analytics.benchmark_master`, candidate tokens must be matched against `sec_financials.presentation_labels pl` on `pl.adsh = nf.adsh`:
```sql
pl.adsh = rcf.accessionNumber
AND (
    pl.tag = rcf.rawMeasure
    OR pl.tag = CONCAT(rcf.rawMeasure, 'Member')
    OR pl.tag = CONCAT('index_', rcf.rawMeasure, 'Member')
    OR pl.tag = CONCAT('rr_', rcf.rawMeasure, 'Member')
)
```
- Label Precedence: `COALESCE(NULLIF(TRIM(pl.terse), ''), NULLIF(TRIM(pl.std), ''), NULLIF(TRIM(pl.verbose), ''), clean_token(rawMeasure))`

---

### [FR-200] Filing Cadence & State Transition Engine

#### FR-201: Point-in-Time Quarterly Cadence Evaluation
Mutual fund benchmarks must reflect what was legally disclosed in that specific quarterly filing cohort. The pipeline must not project past benchmarks indefinitely forward when intermediate quarters omit benchmark disclosures.
- **Scenario A (Current Filing Contains Comparative Benchmark Disclosures):**
  - `fund_master.benchmark_id` = Extracted primary `benchmark_id`.
  - `fund_master.benchmark_name` = Extracted primary `benchmark_name`.
  - `fund_master.reporting_cadence` = `'ANNUAL'`.
  - `fund_master.has_daily_pricing` = `TRUE`.
- **Scenario B (Current Filing Does Not Disclose Comparative Returns / Form 497 Supplement):**
  - If a fund reported a benchmark in Q1 (e.g., S&P 500) but files a filing in Q2/Q3 without return disclosures, its active profile for that filing cohort must reflect the non-reporting state.
  - `fund_master.benchmark_id` = `0`.
  - `fund_master.benchmark_name` = `'Not Disclosed / New Inception'`.
  - `fund_master.reporting_cadence` = `'SUPPLEMENTAL'`.
  - `fund_master.has_daily_pricing` = `FALSE` (Cadence flag toggled to `FALSE` to denote no active return disclosure).
  - This state persists until a subsequent filing containing an active, non-zero return table is ingested.

#### FR-202: Historical Traceability Preservation
Point-in-time filing transitions must not destroy audit records:
- The historical link to prior benchmarks must remain permanently queryable via `analytics.fund_benchmark_association` (keyed by `series_id` and `accession_number`).
- The quarterly presence must remain recorded in `analytics.quarter_fund_association` (linking `quarter_id`, `series_id`, and `accession_number`).

#### FR-203: Downstream API Contract Guarantee (`is_primary`)
Every filing processed must have at least one valid primary record in `fund_benchmark_association`:
- For filings with resolved benchmarks: the primary index is stored with `is_primary = TRUE`.
- For filings lacking benchmarks (new inceptions, exempt money market funds, supplemental updates): a fallback record (`benchmark_id = 0`, `'Not Disclosed / New Inception'`) is written with `is_primary = TRUE` for that filing's `accession_number`.

---

### [FR-300] Investment Strategy Classification Engine

#### FR-301: Strategy Categorization
Funds must be deterministically classified into one of three standard values:
- `ACTIVE`: Discretionary asset selection, bottom-up/top-down fundamental analysis, or research-driven allocation.
- `PASSIVE`: Index tracking, full replication, or representative sampling matching an underlying benchmark.
- `UNKNOWN`: Prospectus disclosure missing, blank, or non-matching against rule patterns.

#### FR-302: Priority-Driven Compliance Scanning
Strategy classification is evaluated in `FundsItemProcessor` using active rules loaded from `analytics.compliance_rules`:
1. Extract narrative disclosure from `sec_financials.text_disclosures` using:
   - Primary: `rr:StrategyNarrativeTextBlock`
   - Secondary: `rr:InvestmentStrategyTextBlock`
2. If narrative is blank or null, assign `UNKNOWN` immediately.
3. Iterate sequentially through active rules ordered by `priority ASC`:
   - **Priority 10–19 (Negations & Exclusions):** Evaluate negative declarations (e.g., `not an index fund`, `does not track`). If matched, override passive terminology.
   - **Priority 20–29 (Core Passive):** Evaluate index tracking patterns (e.g., `tracks the performance of`, `full replication`, `passively managed`). Match returns `PASSIVE`.
   - **Priority 30–39 (Core Active):** Evaluate active management patterns (e.g., `actively managed`, `stock selection`, `fundamental analysis`). Match returns `ACTIVE`.
4. If no rules match: Return `UNKNOWN`.

---

### [FR-400] Regulatory Exemption Boundary Handling

#### FR-401: New Inceptions (< 1 Year History)
- Under SEC Form N-1A Item 4(b)(2), funds in operation for less than one calendar year are legally prohibited from displaying historical performance returns and comparative index tables.
- Setting `benchmark_id = 0` (`"Not Disclosed / New Inception"`) for these funds is canonical domain behavior.

#### FR-402: Money Market Funds (SEC Rule 2a-7)
- Money market instruments disclose 7-day net yields and are exempt from broad-market comparative index requirements.
- Identifying share-class yields without broad-market index rows must result in `benchmark_id = 0` without generating parsing exceptions.

---

## 5. Database Schema & Data Dictionary

### 5.1 `analytics.staged_filing_benchmark` (Fixed 3-Column Bridge)
Stores intermediate accession-to-benchmark resolutions per quarter.

| Column Name | Data Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `accession_number` | `VARCHAR(25)` | PK, NOT NULL | SEC accession identifier (`adsh`) |
| `benchmark_id` | `INT` | PK, NOT NULL, FK $\to$ `benchmark_master` | Canonical benchmark identifier |
| `created_at` | `TIMESTAMPTZ` | DEFAULT CURRENT_TIMESTAMP | Staging record creation timestamp |

### 5.2 `analytics.benchmark_master`
Canonical catalog of all securities market indices and fallback entities.

| Column Name | Data Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `benchmark_id` | `INT` | PK (Identity sequence) | Canonical ID (`0` reserved for Unassigned/New Inception) |
| `benchmark_name` | `TEXT` | NOT NULL, UNIQUE | Human-readable index title |
| `benchmark_provider`| `VARCHAR(64)` | NULLABLE | Provider (S&P, Bloomberg, Russell, etc.) |
| `benchmark_type` | `VARCHAR(32)` | DEFAULT 'Broad Market' | Category (Broad Market, Sector, Unassigned) |
| `accession_number` | `VARCHAR(20)` | NULLABLE | Filing accession provenance |

### 5.3 `analytics.fund_benchmark_association`
Historical 1:N association bridge linking filings to benchmark disclosures.

| Column Name | Data Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `id` | `INT` | PK (Identity sequence) | Synthetic row identifier |
| `series_id` | `VARCHAR(20)` | NOT NULL, FK $\to$ `fund_master` | SEC Series identifier (`S000...`) |
| `benchmark_id` | `INT` | NOT NULL, FK $\to$ `benchmark_master` | Reference to index entity |
| `benchmark_name` | `TEXT` | NULLABLE | Display name of the associated index |
| `is_primary` | `BOOLEAN` | DEFAULT FALSE | `TRUE` if primary benchmark for that accession |
| `accession_number` | `VARCHAR(25)` | NULLABLE | Associated filing accession provenance |

### 5.4 `analytics.fund_master`
Operational master record representing the current status and reporting cadence of each fund.

| Column Name | Data Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `fund_id` | `BIGSERIAL` | PRIMARY KEY | Synthetic surrogate identifier |
| `series_id` | `VARCHAR(10)` | UNIQUE, NOT NULL | SEC Series identifier (`S000...`) |
| `cik` | `INTEGER` | NOT NULL | Central Index Key |
| `fund_name` | `TEXT` | NOT NULL | SEC Series legal name |
| `primary_ticker` | `VARCHAR(10)` | NULLABLE | Primary share class ticker |
| `strategy_type` | `VARCHAR(20)` | NOT NULL | Classified strategy (`ACTIVE`, `PASSIVE`, `UNKNOWN`) |
| `benchmark_id` | `INT` | FK $\to$ `benchmark_master` | Active benchmark ID (`0` if no active disclosure) |
| `benchmark_name` | `TEXT` | NULLABLE | Active benchmark name |
| `reporting_cadence`| `VARCHAR(20)` | NOT NULL | Active cadence (`ANNUAL`, `SUPPLEMENTAL`) |
| `has_daily_pricing`| `BOOLEAN` | NOT NULL | Active cadence toggle: `TRUE` if active return disclosed, `FALSE` if non-disclosed/supplemental |

---

## 6. Verification & Validation Quality Gates

| Test Phase | Verification Target | Acceptance Criteria |
| :--- | :--- | :--- |
| **Extraction Coverage** | Tag & Dimension Reader | 100% of non-money-market funds with return tables in Form 485 filings resolve to a non-zero `benchmark_id`. |
| **Cadence Transition** | Java Processor State Switching | When processing an annual filing followed by a supplemental filing, `reporting_cadence` transitions from `ANNUAL` to `SUPPLEMENTAL`, and `has_daily_pricing` transitions to `FALSE`. |
| **Audit Traceability** | Association Bridge Integrity | `fund_benchmark_association` retains distinct rows per accession for all prior quarters, ensuring historical associations remain queryable. |
| **Regulatory Boundary** | Exemption Validation | Exact isolation of ~2,292 funds (New Inceptions and Rule 2a-7 Money Market Trusts) possessing `benchmark_id = 0` with zero pipeline runtime exceptions. |
| **Strategy Engine** | Narrative Classification | Over 90% of funds with disclosed strategy text blocks successfully classify as `ACTIVE` or `PASSIVE`. |