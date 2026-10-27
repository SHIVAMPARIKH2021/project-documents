```text
[ Incoming RawFundSource (item) ]
               │
               ▼
   Resolve Current Filing Benchmark
 (from accession-level staging cache)
               │
      ┌────────┴────────┐
      ▼                 ▼
[ Benchmark Found ]   [ No Benchmark in Filing ]
      │                 │
      │                 ▼
      │        Check Fund History / Prior Cadence
      │                 │
      │        ┌────────┴────────┐
      │        ▼                 ▼
      │   [ Prior Annual    [ Genuine New Fund /
      │     Track Record ]    Exempt Money Market ]
      │        │                 │
      │        ▼                 ▼
      │   Keep benchmark,   benchmark_id = 0,
      │   set Cadence =     Cadence = ANNUAL,
      │   SUPPLEMENTAL/     is_primary = TRUE
      │   FALSE             (Canonical Fallback)
      │
      ▼
Set fund_master fields,
build associations with strict primary flags,
and pass to FundsItemWriter.
```