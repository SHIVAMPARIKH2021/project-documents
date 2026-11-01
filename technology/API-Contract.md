# SEC EDGAR Fund & Benchmark Data Lake - API Contract & Response Payloads

This specification defines the complete OpenAPI 3.1 contract for the SEC EDGAR mutual fund platform, with concrete, production-grade JSON response schemas and examples for each endpoint.

---

## 1. Complete OpenAPI 3.1 Contract with Inline Response JSONs

```yaml
openapi: 3.1.0
info:
  title: SEC EDGAR Mutual Fund & Benchmark Analytics API
  description: RESTful domain API abstraction over SEC EDGAR Form N-1A and N-PORT data lake disclosures with full response JSON payloads.
  version: 1.0.0
servers:
  - url: https://api.alpha-edgar.internal/api/v1
    description: Internal REST Cluster

paths:
  # =========================================================================
  # 1. FILINGS & BATCH INGESTION
  # =========================================================================
  /filings:
    get:
      summary: Query raw SEC EDGAR filings
      tags: [Filings & Ingestion]
      parameters:
        - name: form_type
          in: query
          schema:
            type: string
            example: "N-PORT,N-1A"
        - name: cik
          in: query
          schema:
            type: string
            example: "0000036405"
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: List of indexed filings in the data lake
          content:
            application/json:
              example:
                total: 45
                page: 1
                limit: 20
                items:
                  - accessionNumber: "0001193125-25-012345"
                    cik: "0000036405"
                    entityName: "Vanguard Index Funds"
                    formType: "N-PORT"
                    filingDate: "2025-03-31"
                    reportDate: "2025-02-28"
                    fileSizeInBytes: 1248920
                    isXBRL: true
                  - accessionNumber: "0001193125-25-012999"
                    cik: "0000036405"
                    entityName: "Vanguard Index Funds"
                    formType: "N-1A"
                    filingDate: "2025-04-15"
                    reportDate: "2024-12-31"
                    fileSizeInBytes: 894000
                    isXBRL: true

  /filings/{accessionNumber}:
    get:
      summary: Retrieve filing metadata
      tags: [Filings & Ingestion]
      parameters:
        - name: accessionNumber
          in: path
          required: true
          schema:
            type: string
            example: "0001193125-25-012345"
      responses:
        '200':
          description: Filing metadata object
          content:
            application/json:
              example:
                accessionNumber: "0001193125-25-012345"
                cik: "0000036405"
                entityName: "Vanguard Index Funds"
                formType: "N-PORT"
                filingDate: "2025-03-31"
                reportDate: "2025-02-28"
                acceptanceDateTime: "2025-03-31T17:30:12Z"
                seriesList: ["S000006037", "S000007812"]
                documentCount: 3
                rawStorageUri: "s3://sec-edgar-lake/2025/03/31/0001193125-25-012345.xml"

  /filings/{accessionNumber}/raw-content:
    get:
      summary: Stream raw XML/XBRL filing payload
      tags: [Filings & Ingestion]
      parameters:
        - name: accessionNumber
          in: path
          required: true
          schema:
            type: string
            example: "0001193125-25-012345"
      responses:
        '200':
          description: Raw XML/XBRL payload stream
          content:
            application/xml:
              schema:
                type: string
                example: |
                  <?xml version="1.0" encoding="UTF-8"?>
                  <edgarSubmission xmlns="http://www.sec.gov/edgar/nport">
                    <headerData>
                      <submissionType>NPORT-P</submissionType>
                      <filerInfo><filerCik>0000036405</filerCik></filerInfo>
                    </headerData>
                  </edgarSubmission>

  /sync-jobs:
    get:
      summary: List batch pipeline sync executions
      tags: [Filings & Ingestion]
      responses:
        '200':
          description: List of batch execution records
          content:
            application/json:
              example:
                total: 2
                items:
                  - jobId: "c82db614-b4cf-4ca6-91e8-76ad06263e8a"
                    formTypes: ["N-PORT"]
                    status: "COMPLETED"
                    startedAt: "2026-03-01T04:00:00Z"
                    completedAt: "2026-03-01T04:18:24Z"
                    recordsIngested: 28592
                    errorCount: 0
                  - jobId: "e93ab720-c7ba-4ba8-81d2-96cb07261a12"
                    formTypes: ["N-1A"]
                    status: "RUNNING"
                    startedAt: "2026-03-02T01:30:00Z"
                    completedAt: null
                    recordsIngested: 1420
                    errorCount: 0
    post:
      summary: Trigger an ad-hoc ingestion sync job
      tags: [Filings & Ingestion]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                formTypes:
                  type: array
                  items:
                    type: string
                cadence:
                  type: string
                  enum: [DAILY, MONTHLY, QUARTERLY]
            example:
              formTypes: ["N-PORT", "N-1A"]
              cadence: "DAILY"
      responses:
        '202':
          description: Ingestion job accepted
          content:
            application/json:
              example:
                jobId: "f71cd932-a5bc-42b7-84bc-51be09374d98"
                status: "PENDING"
                formTypes: ["N-PORT", "N-1A"]
                queuedAt: "2026-03-02T02:00:00Z"
                message: "Ad-hoc SEC EDGAR ingestion pipeline successfully dispatched to worker cluster."

  /sync-jobs/{jobId}:
    get:
      summary: Check status of batch sync job
      tags: [Filings & Ingestion]
      parameters:
        - name: jobId
          in: path
          required: true
          schema:
            type: string
            example: "f71cd932-a5bc-42b7-84bc-51be09374d98"
      responses:
        '200':
          description: Job status and execution progress
          content:
            application/json:
              example:
                jobId: "f71cd932-a5bc-42b7-84bc-51be09374d98"
                status: "RUNNING"
                percentComplete: 64.5
                processedFilings: 840
                totalFilingsTargeted: 1302
                elapsedSeconds: 245
                currentStep: "PARSING_XBRL_SCHEDULE_OF_INVESTMENTS"

  # =========================================================================
  # 2. MASTER FUNDS & REGISTRY
  # =========================================================================
  /funds:
    get:
      summary: Search and list master funds
      tags: [Funds Registry]
      parameters:
        - name: q
          in: query
          schema:
            type: string
            example: "VFIAX"
        - name: asset_class
          in: query
          schema:
            type: string
            enum: [EQUITY, FIXED_INCOME, MULTI_ASSET]
        - name: strategy
          in: query
          schema:
            type: string
            enum: [PASSIVE, ACTIVE]
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 10
      responses:
        '200':
          description: Paginated funds list
          content:
            application/json:
              example:
                total: 12
                page: 1
                limit: 10
                items:
                  - seriesId: "S000006037"
                    ticker: "VFIAX"
                    cusip: "922908710"
                    fundName: "Vanguard 500 Index Fund"
                    family: "Vanguard Group Inc"
                    assetCategory: "Equity"
                    strategy: "PASSIVE"
                    cadence: "ANNUAL"
                    status: "ACTIVE"
                  - seriesId: "S000002157"
                    ticker: "FCNTX"
                    cusip: "316071109"
                    fundName: "Fidelity Contrafund"
                    family: "Fidelity Management & Research"
                    assetCategory: "Equity"
                    strategy: "ACTIVE"
                    cadence: "ANNUAL"
                    status: "ACTIVE"

  /funds/{seriesId}:
    get:
      summary: Retrieve fund profile
      tags: [Funds Registry]
      parameters:
        - name: seriesId
          in: path
          required: true
          schema:
            type: string
            example: "S000006037"
      responses:
        '200':
          description: Fund profile details
          content:
            application/json:
              example:
                seriesId: "S000006037"
                cik: "0000036405"
                primaryTicker: "VFIAX"
                cusip: "922908710"
                fundName: "Vanguard 500 Index Fund"
                family: "Vanguard Group Inc"
                assetCategory: "Equity"
                strategyType: "PASSIVE"
                cadence: "ANNUAL"
                latestAccessionNumber: "0001193125-25-012345"
                primaryBenchmark: "S&P 500 Index"
                shareBalance: "125,480,000"
                marketValueUsd: "$512,450,000,000"
                portfolioWeight: "100.0%"
                netExpenseRatio: 0.0004
                managementFee: 0.0003
                bestQuarter:
                  period: "2020-Q2"
                  returnPercentage: 20.54
                worstQuarter:
                  period: "2020-Q1"
                  returnPercentage: -19.60

  /funds/{seriesId}/holdings:
    get:
      summary: Paginated Form N-PORT holdings for latest period
      tags: [Funds Registry]
      parameters:
        - name: seriesId
          in: path
          required: true
          schema:
            type: string
            example: "S000006037"
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 50
        - name: sort_by
          in: query
          schema:
            type: string
            default: "weight"
      responses:
        '200':
          description: Security-level constituent holdings
          content:
            application/json:
              example:
                seriesId: "S000006037"
                asOfDate: "2025-02-28"
                accessionNumber: "0001193125-25-012345"
                totalHoldings: 503
                page: 1
                limit: 3
                items:
                  - securityTitle: "MICROSOFT CORP"
                    cusip: "594918104"
                    ticker: "MSFT"
                    assetCategory: "EQUITY"
                    balanceShares: "12,410,200"
                    marketValueUsd: "$5,460,488,000"
                    portfolioWeight: 7.15
                  - securityTitle: "APPLE INC"
                    cusip: "037833100"
                    ticker: "AAPL"
                    assetCategory: "EQUITY"
                    balanceShares: "24,800,100"
                    marketValueUsd: "$5,332,021,500"
                    portfolioWeight: 6.98
                  - securityTitle: "NVIDIA CORP"
                    cusip: "67066G104"
                    ticker: "NVDA"
                    assetCategory: "EQUITY"
                    balanceShares: "38,100,500"
                    marketValueUsd: "$4,876,864,000"
                    portfolioWeight: 6.38

  /funds/{seriesId}/analytics/performance:
    get:
      summary: Standardized Item 4b calendar returns
      tags: [Analytics & Risk]
      parameters:
        - name: seriesId
          in: path
          required: true
          schema:
            type: string
            example: "S000006037"
      responses:
        '200':
          description: Historical annual return trajectory
          content:
            application/json:
              example:
                seriesId: "S000006037"
                benchmarkComparison: "S&P 500 Index"
                annualReturns:
                  - year: "2020"
                    returnPercentage: 18.40
                  - year: "2021"
                    returnPercentage: 28.71
                  - year: "2022"
                    returnPercentage: -18.11
                  - year: "2023"
                    returnPercentage: 26.29
                  - year: "2024"
                    returnPercentage: 25.02
                trailingPeriods:
                  oneYear: 25.02
                  threeYearAnnualized: 11.23
                  fiveYearAnnualized: 15.05
                  tenYearAnnualized: 12.85

  /funds/{seriesId}/analytics/risk-metrics:
    get:
      summary: Active share, beta, tracking error vs benchmark
      tags: [Analytics & Risk]
      parameters:
        - name: seriesId
          in: path
          required: true
          schema:
            type: string
            example: "S000006037"
        - name: benchmark_id
          in: query
          schema:
            type: string
            example: "sp500"
      responses:
        '200':
          description: Risk and factor metrics
          content:
            application/json:
              example:
                seriesId: "S000006037"
                benchmarkId: "sp500"
                activeShare: 0.85
                trackingError: 0.12
                beta: 1.00
                rSquared: 0.999
                standardDeviation: 15.2
                sharpeRatio: 1.18

  # =========================================================================
  # 3. COST SIMULATIONS
  # =========================================================================
  /simulations/expense-drags:
    post:
      summary: Calculate 10-year compounding opportunity cost
      tags: [Simulations]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [seriesId, principal, marketReturn, years]
              properties:
                seriesId:
                  type: string
                principal:
                  type: number
                marketReturn:
                  type: number
                years:
                  type: integer
            example:
              seriesId: "S000006037"
              principal: 10000
              marketReturn: 0.07
              years: 10
      responses:
        '200':
          description: Compounding fee drag and trajectory projections
          content:
            application/json:
              example:
                seriesId: "S000006037"
                principal: 10000
                assumedGrossReturn: 0.07
                netExpenseRatio: 0.0004
                statutoryExpenseSchedule:
                  expenseExampleYr01: 4
                  expenseExampleYr03: 13
                  expenseExampleYr05: 23
                  expenseExampleYr10: 51
                tenYearCompoundedEndingValue:
                  grossZeroFeeBenchmark: 19671.51
                  netFundEndingValue: 19577.01
                  compoundingOpportunityCostLost: 94.50
                  percentageLossRatio: 0.48
                annualTrajectory:
                  - year: 1
                    benchmarkValue: 10700.00
                    fundValue: 10696.00
                  - year: 5
                    benchmarkValue: 14025.52
                    fundValue: 13996.11
                  - year: 10
                    benchmarkValue: 19671.51
                    fundValue: 19577.01

  # =========================================================================
  # 4. PORTFOLIOS & REBALANCING
  # =========================================================================
  /portfolios:
    get:
      summary: List user model portfolios
      tags: [Portfolios]
      responses:
        '200':
          description: List of user portfolios
          content:
            application/json:
              example:
                items:
                  - id: "port_1"
                    name: "Retirement Core (60/40)"
                    targetBenchmark: "S&P 500 Index"
                    totalHoldingsCount: 2
                    totalAllocationPercentage: 100
                    weightedExpenseRatio: 0.24
                  - id: "port_2"
                    name: "Aggressive Growth Strategy"
                    targetBenchmark: "Russell 1000 Growth Index"
                    totalHoldingsCount: 2
                    totalAllocationPercentage: 100
                    weightedExpenseRatio: 0.39
    post:
      summary: Create model portfolio
      tags: [Portfolios]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [name]
              properties:
                name:
                  type: string
                targetBenchmark:
                  type: string
            example:
              name: "Global All-Weather Portfolio"
              targetBenchmark: "S&P 500 Index"
      responses:
        '201':
          description: Portfolio created
          content:
            application/json:
              example:
                id: "port_1740960000"
                name: "Global All-Weather Portfolio"
                targetBenchmark: "S&P 500 Index"
                totalAllocationPercentage: 0
                weightedExpenseRatio: 0.00
                holdings: []

  /portfolios/{portfolioId}:
    get:
      summary: Get portfolio allocation and metrics
      tags: [Portfolios]
      parameters:
        - name: portfolioId
          in: path
          required: true
          schema:
            type: string
            example: "port_1"
      responses:
        '200':
          description: Model portfolio details
          content:
            application/json:
              example:
                id: "port_1"
                name: "Retirement Core (60/40)"
                targetBenchmark: "S&P 500 Index"
                totalAllocationPercentage: 100
                weightedExpenseRatio: 0.24
                weightedActiveShare: 17.4
                expectedTrackingError: 3.2
                benchmarkAlignmentPercentage: 89
                holdings:
                  - id: "h1"
                    seriesId: "S000006037"
                    fund: "Vanguard 500 Index Fund"
                    ticker: "VFIAX"
                    weight: 60
                    expenseRatio: 0.04
                    benchmark: "S&P 500"
                  - id: "h2"
                    seriesId: "S000002157"
                    fund: "Fidelity Contrafund"
                    ticker: "FCNTX"
                    weight: 40
                    expenseRatio: 0.55
                    benchmark: "S&P 500"
    put:
      summary: Replace portfolio model configuration
      tags: [Portfolios]
      parameters:
        - name: portfolioId
          in: path
          required: true
          schema:
            type: string
            example: "port_1"
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
            example:
              name: "Retirement Core (60/40) Balanced"
              targetBenchmark: "S&P 500 Index"
              holdings:
                - seriesId: "S000006037"
                  ticker: "VFIAX"
                  weight: 50
                - seriesId: "S000002157"
                  ticker: "FCNTX"
                  weight: 50
      responses:
        '200':
          description: Portfolio updated
          content:
            application/json:
              example:
                id: "port_1"
                name: "Retirement Core (60/40) Balanced"
                targetBenchmark: "S&P 500 Index"
                totalAllocationPercentage: 100
                weightedExpenseRatio: 0.295
                updatedAt: "2026-03-02T02:05:00Z"
    delete:
      summary: Delete portfolio
      tags: [Portfolios]
      parameters:
        - name: portfolioId
          in: path
          required: true
          schema:
            type: string
            example: "port_1"
      responses:
        '204':
          description: Portfolio deleted (No Content)

  # =========================================================================
  # 5. WATCHLISTS & ALERTS
  # =========================================================================
  /watchlists:
    get:
      summary: List all watchlists
      tags: [Watchlists]
      responses:
        '200':
          description: Watchlists list
          content:
            application/json:
              example:
                items:
                  - id: "wl_large_cap"
                    name: "US Large Cap Core Funds"
                    fundsMonitoredCount: 25
                    triggeredAlertsCount: 6
                    newFilingsThisMonth: 14
                  - id: "wl_tech_growth"
                    name: "Tech & Growth Aggressive Strategy"
                    fundsMonitoredCount: 12
                    triggeredAlertsCount: 2
                    newFilingsThisMonth: 5
    post:
      summary: Create a watchlist
      tags: [Watchlists]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                name:
                  type: string
            example:
              name: "Dividend Growth Focus"
      responses:
        '201':
          description: Watchlist created
          content:
            application/json:
              example:
                id: "wl_1740961200"
                name: "Dividend Growth Focus"
                fundsMonitoredCount: 0
                triggeredAlertsCount: 0
                createdAt: "2026-03-02T02:06:00Z"

  # =========================================================================
  # 6. SAVED SCREENERS (WORKBENCH)
  # =========================================================================
  /saved-screeners:
    get:
      summary: List saved screening criteria
      tags: [Screeners]
      responses:
        '200':
          description: Stored screener presets
          content:
            application/json:
              example:
                items:
                  - id: "sc_passive_index"
                    name: "Passive Index Funds"
                    description: "ER <= 0.15% | Equity | Passive Index Strategy"
                    criteria:
                      assetClasses: ["Equity"]
                      strategyTypes: ["PASSIVE"]
                      maxExpenseRatio: 0.0015
                  - id: "sc_low_cost_etfs"
                    name: "Low Cost ETFs"
                    description: "ER <= 0.10% | Passive | AUM > $5B"
                    criteria:
                      assetClasses: ["Equity", "Fixed Income"]
                      strategyTypes: ["PASSIVE"]
                      maxExpenseRatio: 0.0010
                      minAum: 5000000000
    post:
      summary: Save screener criteria
      tags: [Screeners]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                name:
                  type: string
                criteria:
                  type: object
            example:
              name: "Ultra Low Fee Core"
              criteria:
                assetClasses: ["Equity", "Fixed Income"]
                strategyTypes: ["PASSIVE"]
                minExpenseRatio: 0.0000
                maxExpenseRatio: 0.0050
                minAum: 1000000000
                primaryBenchmark: "S&P 500 Index"
      responses:
        '201':
          description: Screener created
          content:
            application/json:
              example:
                id: "sc_1740962400"
                name: "Ultra Low Fee Core"
                criteria:
                  assetClasses: ["Equity", "Fixed Income"]
                  strategyTypes: ["PASSIVE"]
                  minExpenseRatio: 0.0000
                  maxExpenseRatio: 0.0050
                  minAum: 1000000000
                  primaryBenchmark: "S&P 500 Index"
                createdAt: "2026-03-02T02:08:00Z"
```

---

## 2. API Response JSON Quick Reference Table

Below is the exact JSON structure returned for each endpoint:

### A. Raw Filings (`GET /filings`)
```json
{
  "total": 45,
  "page": 1,
  "limit": 20,
  "items": [
    {
      "accessionNumber": "0001193125-25-012345",
      "cik": "0000036405",
      "entityName": "Vanguard Index Funds",
      "formType": "N-PORT",
      "filingDate": "2025-03-31",
      "reportDate": "2025-02-28",
      "fileSizeInBytes": 1248920,
      "isXBRL": true
    }
  ]
}
```

### B. Master Fund Details (`GET /funds/{seriesId}`)
```json
{
  "seriesId": "S000006037",
  "cik": "0000036405",
  "primaryTicker": "VFIAX",
  "cusip": "922908710",
  "fundName": "Vanguard 500 Index Fund",
  "family": "Vanguard Group Inc",
  "assetCategory": "Equity",
  "strategyType": "PASSIVE",
  "cadence": "ANNUAL",
  "latestAccessionNumber": "0001193125-25-012345",
  "primaryBenchmark": "S&P 500 Index",
  "shareBalance": "125,480,000",
  "marketValueUsd": "$512,450,000,000",
  "portfolioWeight": "100.0%",
  "netExpenseRatio": 0.0004,
  "managementFee": 0.0003,
  "bestQuarter": {
    "period": "2020-Q2",
    "returnPercentage": 20.54
  },
  "worstQuarter": {
    "period": "2020-Q1",
    "returnPercentage": -19.60
  }
}
```

### C. N-PORT Constituents (`GET /funds/{seriesId}/holdings`)
```json
{
  "seriesId": "S000006037",
  "asOfDate": "2025-02-28",
  "accessionNumber": "0001193125-25-012345",
  "totalHoldings": 503,
  "page": 1,
  "limit": 3,
  "items": [
    {
      "securityTitle": "MICROSOFT CORP",
      "cusip": "594918104",
      "ticker": "MSFT",
      "assetCategory": "EQUITY",
      "balanceShares": "12,410,200",
      "marketValueUsd": "$5,460,488,000",
      "portfolioWeight": 7.15
    },
    {
      "securityTitle": "APPLE INC",
      "cusip": "037833100",
      "ticker": "AAPL",
      "assetCategory": "EQUITY",
      "balanceShares": "24,800,100",
      "marketValueUsd": "$5,332,021,500",
      "portfolioWeight": 6.98
    }
  ]
}
```

### D. Item 3 Cost Drag Simulation (`POST /simulations/expense-drags`)
```json
{
  "seriesId": "S000006037",
  "principal": 10000,
  "assumedGrossReturn": 0.07,
  "netExpenseRatio": 0.0004,
  "statutoryExpenseSchedule": {
    "expenseExampleYr01": 4,
    "expenseExampleYr03": 13,
    "expenseExampleYr05": 23,
    "expenseExampleYr10": 51
  },
  "tenYearCompoundedEndingValue": {
    "grossZeroFeeBenchmark": 19671.51,
    "netFundEndingValue": 19577.01,
    "compoundingOpportunityCostLost": 94.50,
    "percentageLossRatio": 0.48
  },
  "annualTrajectory": [
    { "year": 1, "benchmarkValue": 10700.00, "fundValue": 10696.00 },
    { "year": 5, "benchmarkValue": 14025.52, "fundValue": 13996.11 },
    { "year": 10, "benchmarkValue": 19671.51, "fundValue": 19577.01 }
  ]
}
```

### E. Model Portfolio Details (`GET /portfolios/{portfolioId}`)
```json
{
  "id": "port_1",
  "name": "Retirement Core (60/40)",
  "targetBenchmark": "S&P 500 Index",
  "totalAllocationPercentage": 100,
  "weightedExpenseRatio": 0.24,
  "weightedActiveShare": 17.4,
  "expectedTrackingError": 3.2,
  "benchmarkAlignmentPercentage": 89,
  "holdings": [
    {
      "id": "h1",
      "seriesId": "S000006037",
      "fund": "Vanguard 500 Index Fund",
      "ticker": "VFIAX",
      "weight": 60,
      "expenseRatio": 0.04,
      "benchmark": "S&P 500"
    },
    {
      "id": "h2",
      "seriesId": "S000002157",
      "fund": "Fidelity Contrafund",
      "ticker": "FCNTX",
      "weight": 40,
      "expenseRatio": 0.55,
      "benchmark": "S&P 500"
    }
  ]
}
```