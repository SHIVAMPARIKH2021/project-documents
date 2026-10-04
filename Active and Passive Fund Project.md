## Better Product Ideas to Level Up the Project

### Instead of only displaying static numbers, you can deliver deeper analytical value with these enhancements:

**Active Share / Tracking Divergence (The "Closet Indexer" Detector):**
- Many active funds charge high fees but closely mimic index returns. Add a metric showing how sharply the fund diverges from its benchmark. A low divergence with high active fees warns users of a **closet index fund**. 

**Why Did It Diverge?" Sector Overlay:**  
- Pull top 10 holdings from SEC Form N-PORT (quarterly portfolio holdings filing, fully public and machine-readable).
- Compare the fund's top holdings against the index's top holdings. This visually proves why the fund is active (e.g., "Contrafund holds 12% in Berkshire Hathaway vs. S&P 500's 1.7%").

**Fee vs. Outperformance Score:**
- Pair performance data with fund expense ratios (available via Form N-1A). Show whether the active manager's past quarterly excess return actually covered their higher expense ratio compared to a passive index fund.

|How to find|File Name|Column Name|Column Value|
|:----------|:--------|:----------|:-----------|
|Fund is a Mutual Fund|sub.tsv|form|485BPOS or 497|
||txt.tsv|tag|Document Type|
||txt.tsv|Value|485BPOS or 497|
||txt.tsv|Version|starts with 'oef'|
---
|How to find|File Name|Column Name|Look for|
|:----------|:--------|:----------|:-------|
|Fund Name|txt.tsv|series, adsh||
|CIK|sub.tsv|adsh||
|SeriesId|txt.tsv|adsh||
|ClassId|txt.tsv|adsh||
|Active/Passive|txt.tsv|series, adsh|Startegy text block parsing|
|Benchmark|num.tsv joined with txt.tsv|adsh, tag, version|Performance table index dimensions/elements|


### Which Tag/Field Verifies the Filing Is a Mutual Fund?
Look in ```sub.tsv``` and check the ```form``` column:
- **Mutual Funds (Open-End Management Investment Companies):**
    - ```form``` will be ```485BPOS``` (Post-Effective Amendment to Form ```N-1A```) or ```497K``` (Summary Prospectus).
    - Filings with forms like ```N-4``` or ```N-6``` are insurance/variable annuity products (as seen in the earlier Principal Life example) and should be excluded.
- **Tag verification in ```tag.tsv``` / ```txt.tsv```:**
    - In ```txt.tsv```, check if ```tag``` = ```DocumentType``` has a ```value``` of ```497K``` or ```485BPOS```.
    - Confirm that ```version``` starts with or references the ```rr``` (Risk/Return) or ```oef``` (Open-End Fund) taxonomy rather than ```vip``` (Variable Insurance Products).

### Strategy Text Block Parsing: How to Do It in ```txt.tsv```?
In ```txt.tsv```, locate the row where:   
- ```tag``` = ```StrategyNarrativeTextBlock```  
- ```series``` = [Your Target Fund Series ID]   

Read the ```value``` column (which holds the plain text narrative stripped of HTML tags).

```text
txt.tsv Record:
├── tag    : StrategyNarrativeTextBlock
├── series : S000004123
└── value  : "The Fund seeks to track the performance of a benchmark index that measures 
              the investment return of large-capitalization stocks..."
```

**Classification Logic (Python Example)**
```python
import re

def classify_strategy(text: str) -> str:
    # 1. Passive / Index pattern
    passive_pattern = re.compile(
        r"(tracks?|tracking|replicates?|replicating|corresponds? to|indexing strategy|"
        r"passive(?:ly)? managed|index(?:-|\s+)based)\b", 
        re.IGNORECASE
    )
    
    # 2. Active pattern
    active_pattern = re.compile(
        r"(actively managed|manager selects|fundamental analysis|research-driven|"
        r"stock selection|seeks capital appreciation by selecting|outperform)", 
        re.IGNORECASE
    )
    
    if passive_pattern.search(text):
        return "Passive"
    elif active_pattern.search(text):
        return "Active"
    return "Unclassified"
```
### Benchmark Following: How to Extract from ```num.tsv``` and ```lab.tsv```?
Mutual funds report their past annual returns in num.tsv for each share class and for each benchmark index. The benchmark index rows are identified using the measure or tag columns.

Step-by-Step Resolution
```text
num.tsv                                                     lab.tsv
┌──────────────────────────────────────────────────┐        ┌────────────────────────────────────────────────┐
│ adsh    : 0001193125-26-012345                   │        │ adsh    : 0001193125-26-012345                 │
│ tag     : IndexNoDeductionForFeesExpensesTaxes   │───────►│ tag     : IndexNoDeductionForFeesExpensesTaxes │
| version : rr/2023 or oef/2026                    |(join)  │ version : rr/2023 or oef/2026                  │
│ measure : SP500IndexMember                       │        │ terse   : "S&P 500 Index"                      │
│ value   : 0.1245 (e.g. 12.45% annual return)     │        │ std     : "S&P 500® Index (reflects no fee...)"│
│ series  : S000004123                             │        └────────────────────────────────────────────────┘ 
└──────────────────────────────────────────────────┘
```

**1. In ```num.tsv```:**
- Filter for rows matching your fund's ```series``` ID.
- Look at the ```tag``` column for either:
    - ```IndexNoDeductionForFeesExpensesTaxes```
    - ```PerformanceTableMarketIndex```
    - ```AnnualReturn``` where ```class``` is empty/NULL and ```measure``` is populated with a benchmark token (e.g., ```sp500Member```).
- Note the ```adsh```, ```tag```, and ```version``` values for those rows.  

**2. In ```lab.tsv```:**
- Filter where ```adsh```, ```tag```, and ```version``` match the values from Step 1.
- The ```terse``` or ```std``` column contains the plain-English name of the index (e.g., ***"S&P 500® Index"***, ***"Russell 2000® Index"***, or ***"Bloomberg U.S. Aggregate Bond Index"***).  

**3. If ```lab.tsv``` is unassigned for that custom tag:**
- Open ```txt.tsv``` and search where ```adsh``` and ```series``` match and ```tag``` is ```IndexNoDeductionForFeesExpensesTaxes``` or ```MarketIndex_Name```.
- Read the index name straight from the ```value``` column.

### 1. Why there is no ```rr/2025``` but Only ```oef/2026```?
The SEC officially replaced the ```rr``` (Risk/Return) taxonomy with the oef (Open-End Fund) taxonomy.
- Historically, mutual funds and ETFs tagged prospectuses under the legacy ```rr``` (Risk/Return Summary) taxonomy (e.g., ```rr/2022```, ```rr/2023```).
- In connection with the SEC's modernization of fund disclosures and Tailored Shareholder Reports (TSR) rules, the SEC merged and superseded the legacy Risk/Return taxonomy into a unified oef taxonomy.
- The legacy ```rr``` taxonomy was retired, and ```oef``` is the active taxonomy for open-end fund prospectuses (Forms ```N-1A```, ```497K```, and ```485BPOS```).

### 2. How to Adjust Your Parser for the oef Taxonomy?
Because your 2026 data uses the ```oef``` taxonomy, look for the following versions and tags:  

**A. Filter by Version**:  
In your files (```txt.tsv```, ```tag.tsv```, ```num.tsv```, ```lab.tsv```), match versions beginning with:  
- ```oef```/ (e.g., ```oef/2025```, ```oef/2026```) 

**B. Equivalent Tag Names Under ```oef```**  
The core concept names remain very similar to rr, but keep an eye out for standard oef elements:
|What You Need|Legacy ```rr``` Tag|Current ```oef``` Tag in ```txt.tsv``` / ```num.tsv```|
|:------------|:------------------|:-----------------------------------------------------|
|Strategy (Active / Passive)|```StrategyNarrativeTextBlock```|```StrategyNarrativeTextBlock``` (or ```InvestmentStrategyTextBlock```)|
|Primary Benchmark Label|```IndexNoDeductionForFeesExpensesTaxes``` / ```PerformanceTableMarketIndex```|```IndexNoDeductionsForFeesExpensesTaxes``` / ```MarketIndex``` / ```BroadBasedIndex```|
|Annual Returns|```AnnualReturn```|```AnnlRtrPct``` or ```AnnualReturn```|
|Average Annual Returns (```1Y```, ```5Y```, ```10Y```)|```AverageAnnualReturnYear01``` / ```05``` / ```10```|```AvgAnnlRtrPct```|

When extracting strategy narrative to classify active vs. passive, filtering for rows where ```tag``` contains ```Strategy``` and ```version``` starts with ```oef``` will give you the exact text block needed from ```txt.tsv```