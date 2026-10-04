# Basics of Active/Passive Funds

## What is Active/Passive funds in U.S. Retirement Plan Solutions?

## Passive Funds
A passive fund aims to **track the perforamnce of a market index** rather than outperform it.

Examples:
- S&P 500 Index Fund
- Russell 2000 Index Fund
- Total Stock Market Index Fund

Characteristics:
- Follows a predefined index.
- Minimal portfolio manager decision-making.
- Typically lower expense reatio.
- Objective is to match, not beat, the benchmark.

## Active Funds
An active fund has portfolio managers making investment decisions to try to **outperform a benchmark.**

Examples:
- Fidelity Contrafund
- American Funds Growth Fund of America
- T. Rowe Price Blue Chip Growth Fund

Characteristics:
- Security selection by fund managers.
- Research-driven investment decisions.
- Usually higher fees.
- Objective is to beat the benchmark.

## Regarding Research Providers and Advisor Approval
**"If fund is not an index fund and approved by the entity providing research to advisors, then is it passive fund?"**

This is **incorrect**

Research firms such as:
- Morningstar
- Callan
- Wilshire
- Mercer
- Envestnet
- AssetMark

may **recommend**, **rate**, or **approve** funds for advisor use. However, their approval does **not change the fund's Active/Passive calssification.**

For example:
|**Fund**|**Index Based?**|**Morningstar Approved?**|**Classification**|
|:-------|:--------------:|:-----------------------:|:----------------:|
|Vanguard 500 Index Fund|Yes|Yes|Passive|
|Fidelity Contrafund|No|Yes|Active|
|American Funds Growth Fund|No|Yes|Active|
|Schwab S&P 500 Index Fund|Yes|No|Passive|

The classification is determined by **investment strategy**, not by who recommends it.

**Be careful**: a fund can be"

- Approved by Morningstar/Mercer/Wilshire
- Included in a advisor model

and still can be **Active**, because the classification is based on its investment management approach, not research approval.

## In Retirement Plan Solutions
Sometimes you will hear additional terms:

- **Passive lineup** = mostly index funds.
- **Active lineup** = mostly actively managed funds.
- **Hybrid/Core lineup** = mix of active and passive funds.
- **Qualified Default Investment Alternatives (QDIAs)** such as Target Date Funds can themselves be active, passive, or blended.

For Example:

- Vanguard Target Retirement Funds -> largely passive.
- American Funds Target Date Series -> largely active.
- Some providers offer blended target date funds.

## Quick Rule
```text
Ask:
|
|       "Is the fund trying to replicate an index or is a manager making investment decisions?"
```
- Replicates index -> **Passive**
- Manager selects investments to outperform benchmark -> **Active**

Advisor approval, research ratings, or inclusion on a recommended list do **not** determine whether a fund ia active or passive.

If you're working in **U.S. retirement plan recordkeeping/advisory platforms**, these are some of the most commonly used **actively managed mutual funds.** (These is no official "Top 10" list, so I've selected large, widely used active funds often found in 401(k) plans.)

|**Fund Name**|**Ticker**|**CUSIP**|
|:------------|:---------|:--------|
|Fiedlity Contrafund|FCNTX|316071109|
|American Funds Growth Fund of America A|AGTHX|399874106|
|American FUnds Washington Mututal A|AWSHX|939330106|
|T. Rowe Price Blue Chip Growth|TRBCX|7795Q106|
|PIMCO Total Return Institutional|PTTRX|693393700|
|Dodge & Cox Stock Fund|DODGX|256219106*|
|Dodge & Cox International Stock|DODFX|256206103*|
|Fidelity Growth Company Fund|FDGRX|316200104*|
|American Funds New Perspective A|ANWPX|649874106*|
|American Funds Investment Company of America A|AIVSX|461308108*|

\* These CUSIPs are commonly referred industry indentifiers, but I'd recommend verifying against plan's security master, Morningstar database, DTCC, or recordkeeper source before using them in production.

## Important for Retirement Platforms
In many retirement systems you'll also encounter:

- **Investment Type** = Active/Passive
- **Asset Class** = Large Cap Growth, Large Value, Bond, etc.

## All the index funds are passive funds? What is Index of a fund?
Yes, generally spieaking.

## What is an Index?
An **Index** is simply a list of securites that represents a perticular market or segment of the market.

Examples:
- **S&P 500 Index** = approximately 500 large U.S. companies
- **Russel 2000 Index** = small-cap U.S. companies
- **NASDAQ-100 Index** = 100 large non-financial complanies listed of Nasdaq
- **MSCI EAFE Index** = developed international markets outside the U.S. and Canada

Think of an Index as a **socreboard or benchmark**. You cannot invest directly in index itself. Instead, fund companies create funds that attempt to replicate the index.

## Why are index funds called Passive?
Suppose the S&P 500 contains:
1. Microsoft
2. Apple
3. Amazon
4. Nvidia
5. Meta
6. ...and hundreds more

A passive S&P 500 fund simply buys those complanies in approximately ***the same weights as the index.*** (***What does **the same weights as the index** means?***)
The manager is **not deciding**:
- "I like Apple more than Microsoft"
- "I think Nvidia will outperform"

They are just following ***the index rules.*** (***What are these **index rules for indexes like Russell 2000 Index or S&P 500 Index?*****)

Therefore:

&#9989; S&P 500 Index Fund = Passive Fund   
&#9989; Russel 2000 Index Fund = Passive Fund     
&#9989; Total Stock Market Index Fund = Passive Fund      

## Does every fund holding 500 american companies mean it's Passive fund?
**No**. 

This is a common misunderstanding.   
Examples:

**Passive**
- Vanguard 500 Index Fund (VFIAX)
- Fidelity 500 Index Fund (FXAIX)
- Schwab S&P 500 Index Fund (SWPPX)

These track the S&P 500. 

**Active**   
A fund manager may also own 300-500 U.S. companies but ***choose them based on research and investment opinions.*** (***Does this mean Fund Manager decided not to follow index rules and rely on research about perticular fund. Example: A fund has shares from a company which is not U.S. based top 500 company but still growing so choose this fund for plan's lineup. Hence this fund will be an Active fund.***) 
Example:  
- Fidelity Contrafund (FCNTX)
- Growth Fund of America (AGTHX)

These are active because managers decide what to buy and sell. 

## Quick Test
```text
|       "Is the fund trying to match an index?"
|
If Yes -> Passive
|
|       "Is a manager trying to beat an index?"
|
If Yes -> Actve
```

## Example from a 401(k) Plan
|**Fund**|**Benchmark Index**|**Active/Passive**|
|:-------|:------------------|:-----------------|
|Fidelity 500 Index (FXAIX)|S&P 500|Passive|
|Vanguard Total Stock Market (VTSAX)|CRSP US Total Market|Passive|
|Fidelity Contrafund (FCNTX)|S&P 500 (benchmark)|Active|
|Growth Fund of America (AGTHX)|S&P 500 (benchmark)|Active|

Notice something important:  
**Both active and passive funds have benchmarks/indexes.** (***How it is going to **outperform it?*****) 

The difference is:
- Passive fund = **tracks** the index.
- Active fund = **uses the index only for comparision** and tries to outperform it.

The distinction is especially important in retirement-plan systems, where a fund may have an **index field/benchmark field** but still be classified as **active**, because the benchmark is only used to measure performance.

## 1. If a fund beats the S&P 500, does the fund become the new benchmark?
**No**. 

A benchmark is chosen independently, it does not automatically changes another fund outperforms it.

For example:
|**Investment**|**5-Year Anual Returns**|
|:-------------|:-----------------------|
|S&P 500 Index|12%|
|Fidelity Contrafund|15%|

Even though Contrafund perfomed better, the benchmark remains the **S&P 500 Index** because that's the market index the fund is compared against.
Think of it like a marathon:  
- The course = benchmark/index
- The runner = fund

If a runner finishes faster than expected, the course doesn't change.  

## 2. Can a fund ever be used as a benchmark?
**Yes, but dliberately.**  

An advisor or plan sponsor might compare one fund against another fund.  

Example:
- Fund A benchmark: S&P 500
- Fund B benchmark: S&P 500
- Advisory may also compare Fund A v/s Fund B

In that case, Fund B is being used as a comparision reference, but it is not automatically becoming the official benchmark. 

## 3. What is Benchmark Data?
For an S&P 500 benchmark, typical data includes:
- 1-Month Return
- Quarter-to-Date Return
- Year-to-Date Return (YTD)
- 1-Year Return
- 3-Year Annualized Return
- 5-Year Annualized Return
- 10-Year Annualized Return
- Since Inception Return

Example:  
|**Metric**|**Fund**|**Benchmark (S&P 500)**|
|:---------|:-------|:----------------------|
|1 Year|14%|12%|
|3 Year|13%|11%|
|5 Year|12%|10%|

This helps determine whether the fund outperfomed or underperfomed the benchmark. 

## 4. Does benchmark dat Include 5-Year Ratio?
It depends what you mean by "ratio". 

Usually benchmark data includes returns such as:
- 1-Year %
- 3-Year %
- 5-Year %
- 10-Year %

It may also include risk statistics:
- Sharpe Ratio
- Sortino Ratio
- Alpha
- Beta
- Standard Deviation
- Information Ratio

For example:
|**Statistics**|**Fund**|**Benchmark**|
|:-------------|:-------|:------------|
|5-Year Return|12%|10%|
|Sharpe Ratio|0.95|0.80|
|Beta|1.05|1.00|

In retirement-plan platforms, when people say **benchmark data**, they usually mean the benchamark's return history accorss different periods (1Y, 3Y, 5Y, 10Y, etc.)

## 5. How this appears in retirement systems?
A fund record may contain:
1. Fund Name: Fidelity Contrafund
2. Ticker: FCNTX
3. Investment Type: Active
4. Benchmark Name: S&P 500 Index
5. Benchmark Return 1Y: 12.0%
6. Benchmark Return 3Y: 11.5%
7. Benchmark Return 5Y: 10.2%
8. Fund Return 1Y: 14.5%

Notice that:
- The fund is **Active**
- The benchmark is **S&P 500**
- The benchmark remains the same even if the fund beats it

## Easy Rule to Remember
- **Index** = A market basket (S&P 500, Russell 2000, MSCI EAFE).
- **Benchmark** = The standard used to evaluate performance (offten an index).
- **Passive Fund** = Tries to match its benchmark/index. 
- **Active Fund** = Tries to beats its benchmark/index. 
- **Outperforming a benchmark does not make the fund the new benchmark.** 

## Questions and Answers

### 1. A passive S&P 500 fund simply buys those complanies in approximately ***the same weights as the index.*** ***What does **the same weights as the index** means?***

Buying companies in the same weights as the index means the fund allocates its money across each company in the exact proportion that company represents in the total index.  
The S&P 500 uses a float-adjusted market-capitalization weighting system.  
This means larger companies get a larger percentage of the fund's assets, while smaller companies get a smaller percentage.  

**How Weight Is Calculated:**
The weight of an individual company in the index is determined by its proportion of the total market value of all 500 companies combined:
```text
Weight of Company A= 
(Total Market Value of All 500 Companies combined / Market Value of Company A’s Available Shares) × 100
```
If the entire S&P 500 has a combined market value of $50 trillion, and Microsoft accounts for $3.5 trillion of that, its weight in the index is 7%.  
If Apple accounts for $3.0 trillion, its weight is 6%.  
A company near the bottom of the list might be worth $20 billion, representing a weight of just 0.04%.  

**How the Fund Applies This Rule?**    
When you invest money into a passive S&P 500 index fund, the fund does not split the money equally across 500 companies (which would be 0.20% per company). Instead, it mirrors those calculated percentages:
|**Company**|**Example Index Weight**|**Allocation of a $10,000 Investment**|
|:----------|:-----------------------|:-------------------------------------|
|Microsoft|7.0%|$700|
|Apple|6.0%|$600|
|Nvidia|6.0%|$600|
|Amazon|3.5%|$350|
|...495 other companies|Varying (0.01% ~ 3%)|Reminder|
|Total|100%|$10,000|

### 2. What Are the Index Rules for the S&P 500 and Russell 2000?
An index is governed by a strict, published rulebook (methodology) maintained by an index committee or provider (such as S&P Dow Jones Indices or FTSE Russell).  
A passive index fund must follow these rules mechanically:  

**S&P 500 Index Rules:**  
- **Market Capitalization:** Eligible companies must meet a minimum unadjusted market cap threshold (typically >$18–20 billion).  
- **Profitability / Financial Viability:** The company must have positive as-reported earnings over the most recent quarter and cumulatively over the trailing four quarters.  
- **Liquidity & Domicile:** Must be a U.S. company with an active listing on an eligible U.S. exchange, having high trading volume and at least 50% of shares available to the public (public float).  
- **Selection Process:** A discretionary index committee reviews additions and deletions quarterly to ensure balanced sector representation.  

**Russell 2000 Index Rules:** 
- **Market Capitalization Segment:** Captures the small-cap segment by ranking the largest ~3,000 U.S. stocks and taking ranks 1,001 through 3,000.  
- **Rank-Based & Reconstitution:** Unlike the S&P 500's committee, the Russell 2000 is largely formulaic and rank-based. It undergoes an annual "reconstitution" (every June), recalculating eligible ranks and float weights.  
- **No Profitability Filter:** Companies do not need to be profitable to be included; they only need to meet market cap, liquidity, and domicile requirements.

### 3. Does an Active Fund Manager Ignore Index Rules to Rely on Research?
Yes. In an active fund, the portfolio manager is not bound by index composition rules.  
- **Mandate vs. Index:** Active managers follow a prospectus mandate (e.g., "invest at least 80% in large-cap growth equities"), not the rulebook of the benchmark index.  
- **Deviation by Design:** If a manager identifies an international company, a mid-cap stock, or an unlisted/unprofitable company with high growth potential, they can allocate capital to it if their fund's prospectus allows.  
- **Role of the Benchmark:** The S&P 500 or Russell 2000 serves strictly as a yardstick (the "scoreboard") to evaluate whether the manager's independent choices generated excess value.  

### 4. How Does an Active Fund Attempt to Outperform Its Benchmark?
Active managers attempt to generate positive alpha (excess return relative to the benchmark) using three primary levers:
- **Security Selection (Stock Picking):**
    - **Overweighting:** Holding a higher percentage of high-conviction companies than the benchmark holds (e.g., 8% in Nvidia when the index holds 6%).
    - **Underweighting / Excluding:** Holding less of, or completely avoiding, companies they project will lag or deteriorate—even if those companies make up a major slice of the index.

- **Off-Benchmark Bets:** 
    - Buying securities outside the index universe (e.g., foreign equities, private placements, or small/mid-cap growth companies) that offer upside the benchmark cannot access.  

- **Sector Allocation & Cash Management:** 
    -  Shifting capital toward outperforming sectors (e.g., overweighting Technology while underweighting Utilities) based on macroeconomic research.
    - Holding cash during severe market drawdowns to cushion downside risk, whereas passive index funds must stay nearly 100% invested at all times.

## Key Takeaways
**Automatic Rebalancing:** As stock prices move daily, company valuations change. Because market-cap weighting naturally tracks total valuation changes, the fund rarely needs to trade just to maintain market-cap proportions.  
**No Opinion:** The fund manager does not say "I think Nvidia will grow faster, so let's make it 10%". If the index rule dictates 6%, the fund holds approximately 6%.  
**Approximately:** The note says "approximately the same weights" because mutual funds and ETFs must manage everyday realities like small cash buffers for redemptions, dividend distributions, and tiny price variations during trading (tracking error).

### 5. Can One Fund Have More Than One Benchmark?
Yes. Funds frequently report multiple benchmarks in their prospectuses:
- **Primary Benchmark (Broad-Based Market Index):** 
    - Required by SEC Rule 498 / Form N-1A to reflect the overall market in which the fund invests (e.g., S&P 500 Index or Russell 2000 Index).
- **Secondary / Style Benchmark:** 
    - A narrower index matching the manager’s exact focus (e.g., Russell 1000 Growth, NASDAQ-100, or Bloomberg U.S. Aggregate Bond).
- **Blended Benchmark:** 
    - A custom weighted composite of two or more indexes (e.g., 60% S&P 500 / 40% Bloomberg Agg).  

When parsing, you will often find multiple index records for a single fund; the first listed is standardly the primary broad-based benchmark.