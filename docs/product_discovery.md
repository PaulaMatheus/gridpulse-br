# Product Discovery

## GridPulse BR

GridPulse BR is a data product designed to transform public Brazilian
electricity regulatory data into actionable information about electricity
distribution performance.

The project is built incrementally, starting with a small analytical
prototype and evolving toward a production-like data platform using
Databricks.

---

## 1. Problem Statement

Brazilian electricity distribution data is publicly available through
regulatory datasets, but answering business questions often requires
combining, cleaning and transforming data from multiple sources.

An analyst interested in service quality must understand regulatory
indicators, compare actual performance against regulatory limits and
analyze performance across distributors and time periods.

GridPulse BR aims to provide a trusted analytical layer that makes these
analyses easier and reproducible.

---

## 2. Target User

### Energy Performance Analyst

A professional responsible for monitoring electricity distribution
performance and identifying distributors or regions that require further
investigation.

The analyst needs to quickly understand:

- Which distributors are exceeding regulatory service-quality limits?
- Which indicators are deteriorating over time?
- Where should further investigation be prioritized?
- How does performance compare across distributors and periods?

---

## 3. Product Hypothesis

If regulatory and operational electricity data are consolidated into a
trusted analytical data product, analysts can identify deteriorating
distributor performance faster and prioritize investigation.

---

## 4. Prototype Business Question

The first iteration of GridPulse BR will answer one primary question:

> Which electricity distributors are currently showing the worst service
> quality performance?

The prototype will initially use the DEC and FEC service continuity
indicators published by ANEEL.

---

## 5. Initial Metrics

### DEC

Equivalent Duration of Interruption per Consumer Unit.

Measures the average duration of electricity interruptions experienced by
consumers.

### FEC

Equivalent Frequency of Interruption per Consumer Unit.

Measures the average frequency of electricity interruptions experienced by
consumers.

The analysis will compare actual indicators against their regulatory limits.

Initial derived metrics include:

- DEC variance from regulatory limit
- FEC variance from regulatory limit
- DEC limit compliance
- FEC limit compliance

---

## 6. Prototype Scope

### In Scope

- Ingest public DEC/FEC data
- Preserve raw source data
- Clean and standardize relevant fields
- Build analytics-ready service-quality metrics
- Compare actual indicators with regulatory limits
- Rank distributor performance
- Analyze results by period

### Out of Scope

The following capabilities are intentionally excluded from the prototype:

- Energy losses
- Electricity tariffs
- Predictive models
- Machine learning
- Power BI
- Advanced monitoring
- Incremental ingestion

These capabilities may be introduced in later product iterations when they
provide additional value to the user.

---

## 7. Prototype Success Criteria

The prototype will be considered successful when an Energy Performance
Analyst can:

1. Select a reporting period.
2. Compare DEC and FEC against their regulatory limits.
3. Identify distributors exceeding those limits.
4. Rank distributors according to service-quality performance.
5. Identify where further investigation should be prioritized.

---

## 8. Initial Architecture

The prototype will follow a Medallion Architecture implemented with
Databricks and Delta Lake.

```text
ANEEL Open Data
       |
       v
   Ingestion
       |
       v
    Bronze
 Raw DEC/FEC
       |
       v
    Silver
Cleaned and
validated data
       |
       v
     Gold
Distributor quality
data product
       |
       v
 Databricks SQL
       |
       v
Quality Ranking
