### Data Relations and Flow
```mermaid
flowchart TD

    subgraph Sources [1. SEC and Ingestion Sources]
        SRC_JSON["company_tickers_mf.json\n(cik, seriesId, classId, symbol)"]
        SRC_SUB["sub.tsv\n(adsh, cik, name, form, filed)"]
        SRC_TXT["txt.tsv\n(adsh, series, tag, ddate, value)"]
        SRC_NUM["num.tsv\n(adsh, series, tag, version, value)"]
        SRC_LAB["lab.tsv\n(adsh, tag, version, terse, std)"]
    end

    subgraph Engine [2. Spring Batch Transformation Engine]
        ENG_TICKER["Ticker and Class Aggregator\nMaps classId to seriesId and primaryTicker"]
        ENG_CLASS["Active or Passive Classifier\nParses StrategyNarrativeTextBlock"]
        ENG_BM["Benchmark Normalizer\nResolves tags via lab.tsv"]
        ENG_PERF["Performance and Return Processor\nExtracts returns and computes Alpha"]
    end

    subgraph Analytics [3. Analytics Schema PostgreSQL Tables]
        TBL_CLASSES["analytics.fund_classes\n(class_id PK, series_id FK, ticker)"]
        TBL_MASTER["analytics.fund_master\n(series_id PK, cik, fund_name, strategy_type, benchmark_id FK)"]
        TBL_BM["analytics.benchmark_master\n(benchmark_id PK, benchmark_name, benchmark_provider)"]
        TBL_PERF["analytics.fund_performance\n(series_id FK, record_date, fund_return_pct, alpha_pct)"]
    end

    subgraph UI [4. UI Screens and Views]
        PAGE_SEARCH["Page 1: Fund Search Grid\nprimary_ticker, fund_name, strategy_type, benchmark"]
        PAGE_OVERVIEW["Page 2: Fund Profile and Strategy\nfund_family, narrative snippet, tickers, benchmark link"]
        PAGE_BM["Page 3: Benchmark Guidelines\nbenchmark_name, provider, total funds"]
        PAGE_PERF["Page 4: Performance Tracking\nfund_return, benchmark_return, alpha spread"]
    end

    SRC_JSON --> ENG_TICKER
    SRC_SUB --> ENG_CLASS
    SRC_TXT --> ENG_CLASS
    SRC_NUM --> ENG_BM
    SRC_LAB --> ENG_BM
    SRC_NUM --> ENG_PERF

    ENG_TICKER --> TBL_CLASSES
    ENG_TICKER --> TBL_MASTER
    ENG_CLASS --> TBL_MASTER
    ENG_BM --> TBL_BM
    ENG_BM --> TBL_MASTER
    ENG_PERF --> TBL_PERF

    TBL_CLASSES --> TBL_MASTER
    TBL_MASTER --> TBL_BM

    TBL_MASTER --> PAGE_SEARCH
    TBL_MASTER --> PAGE_OVERVIEW
    TBL_CLASSES --> PAGE_OVERVIEW
    TBL_BM --> PAGE_BM
    TBL_PERF --> PAGE_PERF
```

## 1. SEC Form Matrix by Fund Type
| Fund Type | Registration & Prospectus Form (`sub.tsv: form`) | Holdings / Census Form | Reporting Cadence | Primary SEC Dataset / Source Table | How to Derive & Identify |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Open-End Mutual Funds (OEFs)** | • **`485BPOS`** (Annual post-effective update)<br>• **`485APOS`** (Material strategy change)<br>• **`497` / `497K`** (Prospectus supplements/stickers) | • **`N-PORT`** (Monthly holdings filed quarterly)<br>• **`N-CSR` / `N-CSRS`** (Semi-annual reports)<br>• **`N-CEN`** (Annual census) | **Annual** (Prospectus) + **Quarterly** (Holdings) | • `sec_financials.submissions`<br>• `sec_financials.company_tickers_mf`<br>• `sec_financials.text_disclosures` | `form IN ('485BPOS', '497')` AND joined via `series_id` to `company_tickers_mf`. Ticker ends in 'X' (5 letters) or has retail/institutional share classes (Class A, C, I). |
| **Exchange-Traded Funds (ETFs - Open-End)** | • **`485BPOS`** (Registered under Form N-1A)<br>• **`485APOS`**<br>• **`497`** | • **`N-PORT`**<br>• **`N-CEN`**<br>• Daily AP creation/redemption files | **Daily** (Holdings/Baskets) + **Annual** (Prospectus) | • `sec_financials.submissions`<br>• `sec_financials.company_tickers_mf`<br>• `sec_financials.text_disclosures` | `form IN ('485BPOS', '497')` where fund name or share class contains `'ETF'`, or `StrategyNarrativeTextBlock` contains creation unit/authorized participant language. |
| **Exchange-Traded Funds (ETFs - UIT Structure)** | • **`S-6`** (Securities Act of 1933)<br>• **`424B3` / `424B5`** (Prospectus amendments) | • **`N-CEN`**<br>• Daily basket creation files | **Daily** (Baskets) + **Static** (Portfolio) | • SEC EDGAR Corporate / Trust filings | Special cases like SPY (SPDR S&P 500 ETF Trust) or QQQ; filed under Unit Investment Trust rules, not Form N-1A. |
| **Closed-End Funds (CEFs)** | • **`N-2`** (Initial registration statement)<br>• **`N-2/A`** (Pre-effective amendment)<br>• **`N-2ASR`** (Automatic shelf registration) | • **`N-PORT`** (Holdings)<br>• **`N-CSR` / `N-CSRS`** (Semi-annual report)<br>• **`N-CEN`** (Annual census) | **Semi-Annual / Annual** (Filings) + **Daily** (Exchange Trading) | • EDGAR Form N-2 dataset<br>• `sec_financials.submissions` | `form LIKE 'N-2%'`. Absent from `company_tickers_mf` because they do not have multi-class series identifiers (`series_id` / `class_id`). |
| **Unit Investment Trusts (UITs - Traditional)** | • **`S-6`** (Initial filing)<br>• **`N-8B-2`** (Investment Company Act 1940 filing)<br>• **`487`** (Pricing amendment) | • **`N-CEN`** | **Fixed / Life-of-Trust** (Static basket; no ongoing active trading) | • EDGAR Form S-6 dataset<br>• `sec_financials.submissions` | `form IN ('S-6', 'N-8B-2')`. Zero or near-zero portfolio turnover rate; fixed termination date. |
| **Money Market Funds (MMFs)** | • **`485BPOS` / `N-1A`** (Governed by Rule 2a-7)<br>• **`N-CR`** (Material credit events) | • **`N-MFP` / `N-MFP2`** (Monthly portfolio schedule)<br>• **`N-CEN`** | **Daily** (Shadow NAV) + **Monthly** (Portfolio Holdings) | • SEC N-MFP dataset<br>• `sec_financials.submissions` | `form IN ('N-MFP', 'N-MFP2')` or N-1A filings where fund strategy explicitly references Rule 2a-7, WAM (Weighted Average Maturity), or Stable NAV. |
| **Business Development Companies (BDCs)** | • **`N-2`** (Registration statement)<br>• **`N-54A`** (Election to be regulated as BDC) | • **`10-K`** (Annual report)<br>• **`10-Q`** (Quarterly report)<br>• **`8-K`** (Current events) | **Quarterly** (10-Q) + **Annual** (10-K) | • SEC Public Company (10-K/10-Q) datasets | Regulated under Sections 54 through 65 of the 1940 Act; report on standard operating company forms (`10-K`, `10-Q`) rather than `N-PORT`. |

### Form Type Groupings & Purpose Summary
# SEC Form Taxonomy

| Category | SEC Forms | Description / Coverage |
|---|---|---|
| **Fund Prospectuses**<br>*(Annual Mandate/Strategy)* | `485BPOS`, `485APOS`, `497`, `497K` | → Open-End Funds, Open-End ETFs |
| **Closed-End Fund Filings**<br>*(Equity-like Capital)* | `N-2`, `N-2/A`, `N-2ASR` | → Closed-End Funds, BDCs |
| **Portfolio Holdings**<br>*(Quarterly/Monthly Assets)* | `N-PORT` *(Monthly data filed quarterly)*<br>`N-MFP2` *(Money Market Funds monthly)* | Portfolio holdings and asset-level reporting |
| **Unit Investment Trusts**<br>*(Static Baskets)* | `S-6`, `N-8B-2` | → UITs, UIT-structured ETFs |
| **Census & Operations**<br>*(Audits & Master Metadata)* | `N-CEN` *(Annual operations census)*<br>`N-CSR` *(Annual/Semi-annual shareholder report)* | Operational, census, and shareholder reporting |

## Summary

- **Fund Prospectuses** → Annual mandate and investment strategy
- **Closed-End Fund Filings** → Equity-like capital structures
- **Portfolio Holdings** → Periodic portfolio/asset-level data
- **Unit Investment Trusts** → Static investment baskets
- **Census & Operations** → Fund operations, census, and shareholder reporting


## Fund v/s Share Class v/s Benchmark in SEC/EDGAR ```N-1A Form``` Filings
|Entity:|What It Is:|SEC Identifier:|Key Characteristic:|
|-------|-----------|---------------|-------------------|
|Fund (Series)|"The distinct pooled investment vehicle that holds the actual basket of securities (stocks, bonds, cash). All classes share the same manager, portfolio, and strategy."|series_id (S0000xxxxx)|"Investment objective| strategy narrative, total portfolio value."|
|Share Class (Contract)|"A specific ownership slice of the fund tailored to different investor types. Each class has distinct fee structures, minimum investments, tickers, and expense ratios."|class_id (C0000xxxxx)|"Ticker symbol, 12b-1 fees, gross/net expense ratios, load waivers."|
|Comparative Market Index|"An external, unmanaged benchmark against which the fund's performance is measured under SEC Item 4 regulatory disclosure rules."|Dimensional Axis Member (measure)|"Not owned by the fund; reflects no deduction for fees, expenses, or taxes (e.g., Russell 2000, S&P 500)."|

---

# Business Domains of Project
***Domain-Driven Design (DDD) Clarification: Fund vs. Benchmark***
- In a strict Domain-Driven Design model, Fund is an Aggregate Root, but Benchmark is its own independent Aggregate Root, not merely a child/internal entity of Fund.

***Why Benchmark is its own Aggregate Root:***
- A Benchmark (such as the S&P 500 Index or Russell 1000) exists independently in financial markets. It is not owned exclusively by one fund. Multiple funds track the same benchmark simultaneously (e.g., both Vanguard 500 and Fidelity Contrafund reference the S&P 500 Index). If a fund is deleted or delisted, the benchmark continues to exist.   

***How Fund relates to Benchmark:***
- The Fund aggregate references Benchmark via an Association Entity (FundBenchmarkAssociation / BenchmarkMapping) which defines the relationship metadata (e.g., isPrimary: true/false, trackingRole, regulatory disclosures).

```plaintext
========================================================================================
           DOMAIN-DRIVEN DESIGN (DDD) AGGREGATE BOUNDARY GRAPH (PROTOTYPE TABS 1 - 5)
========================================================================================

+--------------------------------------------------------------------------------------+
| [AGGREGATE ROOT] : Fund (Entity)                                                     |
| Identity: seriesId (e.g., S000006037)                                                |
+--------------------------------------------------------------------------------------+
  |
  +-- [ENTITY] ShareClass / Ticker
  |     Identity: classId / ticker (e.g., VFIAX, C000016480)
  |     Attributes: tickerSymbol, inceptionDate, currency, isActive
  |
  +-- [VALUE OBJECT] FundIdentity
  |     Attributes: cik, fundName, fundFamilyName, accessionNumber
  |
  +-- [VALUE OBJECT] StrategyMandate
  |     Attributes: strategyType (ACTIVE | PASSIVE), assetClass, cadence
  |
  +-- [VALUE OBJECT] StatutoryFeeSchedule (Form N-1A Item 3)
  |     Attributes: managementFee, distribution12b1Fee, otherExpenses, netExpenseRatio
  |
  +-- [VALUE OBJECT] ProspectusNarratives
  |     Attributes: investmentObjective, principalStrategyNarrative
  |
  +-- [ENTITY] ExtremeQuarterPerformance (Item 4b)
  |     Attributes: bestQuarterReturn, bestQuarterPeriod, worstQuarterReturn, worstQuarterPeriod
  |
  +-- [ENTITY COLLECTION] CalendarYearReturns (Item 4b History)
  |     Attributes: year, totalReturnPercentage
  |
  +-- [ENTITY] FundBenchmarkAssociation (Bridge / Reference)
  |     Identity: fundBenchmarkAssociationId
  |     References: BenchmarkId (Foreign Aggregate Identity)
  |     Attributes: isPrimary (Boolean), trackingRole, comparativeMetrics
  |
  +-- [VALUE OBJECT] ComparativeBenchmarkMetrics (Snapshot per Fund)
        Attributes: benchmarkExpenseRatioDrag, aumCoverageRatio, ltdActiveShareRatio,
                    trailingReturn1Yr, trailingReturn3Yr, trailingReturn5Yr


+--------------------------------------------------------------------------------------+
| [AGGREGATE ROOT] : Benchmark (Entity)                                                |
| Identity: benchmarkId (e.g., BM_SP500_INDEX)                                         |
+--------------------------------------------------------------------------------------+
  |
  +-- [VALUE OBJECT] BenchmarkMetadata
  |     Attributes: benchmarkName, provider (S&P Dow Jones, FTSE Russell, CRSP), marketType
  |
  +-- [ENTITY COLLECTION] BenchmarkHistoricalPerformance
        Identity: performanceRecordId
        Attributes: asOfDate, return1Yr, return3YrAnn, return5YrAnn, returnLifetime


+--------------------------------------------------------------------------------------+
| [AGGREGATE ROOT] : PortfolioSnapshot (Form N-PORT Monthly Ingestion)                 |
| Identity: snapshotId (e.g., NPORT_S000006037_20250331)                               |
+--------------------------------------------------------------------------------------+
  |
  +-- References: FundId (Foreign Aggregate Identity)
  |
  +-- [VALUE OBJECT] ReportingPeriod
  |     Attributes: periodEndDate, filingDate, accessionNumber
  |
  +-- [ENTITY COLLECTION] PositionHolding (Child Entity)
  |     Identity: holdingId / cusip (e.g., 594918104 - MSFT)
  |     Attributes: securityTitle, ticker, assetCategory, balanceShares, marketValueUsd,
  |                 portfolioWeight, cumulativeConcentrationRank
  |
  +-- [VALUE OBJECT] ConcentrationMetrics
  |     Attributes: totalPositionsCount, top10ConcentrationRatio, weightedAvgMarketCapUsd,
  |                 activeShareRatio
  |
  +-- [VALUE OBJECT COLLECTION] SectorAllocationDistribution
  |     Attributes: gicsSector (Technology, Financials, Healthcare), weightPercentage
  |
  +-- [VALUE OBJECT COLLECTION] RegionalExposureDistribution
  |     Attributes: regionName (United States, Europe, Asia-Pacific), weightPercentage
  |
  +-- [VALUE OBJECT COLLECTION] CurrencyExposureDistribution
  |     Attributes: currencyCode (USD, EUR, GBP), allocationPercentage
  |
  +-- [VALUE OBJECT COLLECTION] MarketCapSpectrumDistribution
  |     Attributes: capTier (Mega, Large, Mid, Small), weightPercentage
  |
  +-- [VALUE OBJECT COLLECTION] LiquidityBuckets (SEC Rule 22e-4)
  |     Attributes: bucketType (Highly Liquid, Moderately Liquid, Less Liquid), weightPercentage
  |
  +-- [VALUE OBJECT] RiskClassification
        Attributes: notionalDerivativesExposure, shortBorrowingLeverageStatus


+--------------------------------------------------------------------------------------+
| [AGGREGATE ROOT] : CostSimulationModel (Transient Simulation Session)                |
| Identity: simulationId (e.g., SIM_VFIAX_10000_7PCT)                                  |
+--------------------------------------------------------------------------------------+
  |
  +-- References: FundId (Foreign Aggregate Identity)
  |
  +-- [VALUE OBJECT] SimulationParameters
  |     Attributes: initialPrincipalUsd, assumedAnnualGrowthRatePercentage, timeHorizonYears
  |
  +-- [VALUE OBJECT] StandardizedProspectusCostTable (Item 3 Algorithm)
  |     Attributes: costYear01, costYear03, costYear05, costYear10
  |
  +-- [VALUE OBJECT COLLECTION] CompoundingWealthTrajectory
  |     Attributes: yearIndex (Yr 0 to Yr 10), zeroFeeBaselineBalance,
  |                 fundAccumulatedBalance, peerAverageAccumulatedBalance
  |
  +-- [VALUE OBJECT] OpportunityCostMetrics
  |     Attributes: totalLostOpportunityDollarValue, reductionInPotentialWealthPercentage
  |
  +-- [VALUE OBJECT COLLECTION] ComparativeFeeScenarios
        Attributes: strategyType, expenseRatio, balanceAfter5Years, balanceAfter10Years,
                    totalFeesIncurred10Years
```

|Tab|Primary Aggregate Root|Associated / Secondary Aggregate Roots|Key Child Entities / Value Objects|
|:----|:----------------------|:--------------------------------------|:----------------------------------|
|Tab 1: Fund Search|Fund|PortfolioSnapshot (Holdings), Benchmark|FundIdentity (Series, Ticker, CUSIP, Name), StrategyMandate (Asset Category, Strategy, Cadence), PositionHolding (via PortfolioSnapshot), FundBenchmarkAssociation|
|Tab 2: Fund Overview|Fund|Benchmark|StatutoryFeeSchedule, ProspectusNarratives, CalendarYearReturns, ExtremeQuarterPerformance|
|Tab 3: Holdings (N-PORT)|PortfolioSnapshot|Fund (via seriesId reference)|PositionHolding|
|Tab 4: Portfolio Analytics|PortfolioSnapshot|Benchmark (for Active Share)|ConcentrationMetrics, SectorAllocationDistribution,RegionalExposureDistribution, MarketCapSpectrumDistribution, LiquidityBuckets|
|Tab 5: Cost Simulation|CostSimulationModel|Fund (source fee rates)|StandardizedProspectusCostTable, CompoundingWealthTrajectory, OpportunityCostMetrics|