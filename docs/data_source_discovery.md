# Data Source Discovery

## GridPulse BR

This document describes the initial data source assessment for the
GridPulse BR prototype.

The goal of this discovery phase is to understand the structure,
granularity, update frequency and relevant attributes of the source data
before defining the ingestion and transformation architecture.

---

## 1. Data Provider

**Provider:** ANEEL — Brazilian Electricity Regulatory Agency

**Dataset:** Collective Continuity Indicators (DEC and FEC)

**Source:** ANEEL Open Data Portal

**Official Dataset:**
https://dadosabertos.aneel.gov.br/dataset/indicadores-coletivos-de-continuidade-dec-e-fec

ANEEL publishes regulatory and operational datasets related to the
Brazilian electricity sector through its Open Data Portal.

For the first GridPulse BR prototype, the Collective Continuity Indicators
dataset was selected because it provides historical information about
electricity service quality across Brazilian electricity distributors.

---

## 2. Dataset Purpose

The dataset provides information about electricity service continuity in
Brazil.

Two of the main indicators used in the GridPulse BR prototype are:

### DEC — Equivalent Duration of Interruption per Consumer Unit

DEC measures the average duration of electricity interruptions experienced
by consumers during a specific period.

### FEC — Equivalent Frequency of Interruption per Consumer Unit

FEC measures the average frequency of electricity interruptions experienced
by consumers during a specific period.

These indicators can be compared with regulatory limits established by
ANEEL to evaluate service-quality performance.

---

## 3. Available Resources

The ANEEL dataset is composed of multiple resources rather than a single
table.

Relevant resources identified during the discovery phase include:

### Service Quality Indicators

Contains the observed continuity indicators reported for electricity
consumer-unit sets.

The available historical data is divided into different periods, including:

- 2010–2019
- 2020–2029

This separation must be considered when designing historical ingestion.

---

### Regulatory Limits

Contains regulatory limits defined for continuity indicators.

These limits allow observed DEC and FEC values to be compared against the
expected regulatory thresholds.

This resource will be required to determine whether service-quality
performance is within or above regulatory limits.

---

### Indicator Domain

ANEEL also provides a reference dataset containing indicator codes and
their corresponding descriptions.

This resource can be used as reference data instead of embedding indicator
descriptions directly into transformation logic.

---

### Additional Resources

The dataset also provides information such as:

- consumer-unit set attributes;
- compensation information;
- additional continuity indicators.

These resources are not required for the first prototype and may be
evaluated in future iterations.

---

## 4. Data Coverage

**Geographical coverage:** Brazil

**Temporal coverage:** Historical data available from 2010 onward

**Temporal granularity:** Monthly

**Update frequency:** Monthly

The recurring publication of new information makes this dataset suitable
for the future implementation of incremental ingestion.

The prototype will initially focus on historical ingestion and analytical
validation before incremental processing is introduced.

---

## 5. Data Formats

The ANEEL Open Data Portal provides resources in formats such as:

- CSV
- Parquet
- ZIP

The appropriate source format for the ingestion pipeline will be selected
after data profiling and evaluation of the available resources.

The choice will consider:

- ingestion reliability;
- schema consistency;
- file size;
- historical availability;
- compatibility with Databricks;
- ability to support future incremental processing.

---

## 6. Initial Relevant Fields

Initial inspection identified attributes similar to the following in the
continuity indicator resources:

| Field | Purpose |
|---|---|
| `DatGeracaoConjuntoDados` | Dataset generation date |
| `SigAgente` | Electricity distributor identifier |
| `NumCNPJ` | Distributor registration identifier |
| `IdeConjUndConsumidoras` | Consumer-unit set identifier |
| `DscConjUndConsumidoras` | Consumer-unit set description |
| `SigIndicador` | Continuity indicator code |
| `AnoIndice` | Indicator year |
| `NumPeriodoIndice` | Indicator reporting period |
| `VlrIndiceEnviado` | Reported indicator value |

The definitive schema will be documented after the source files are
profiled in Databricks.

---

## 7. Data Granularity

One important discovery is that the source data is not necessarily
aggregated directly at distributor level.

Continuity indicators are reported for consumer-unit sets associated with
electricity distributors.

Conceptually:

    Distributor
        |
        +-- Consumer Unit Set A
        +-- Consumer Unit Set B
        +-- Consumer Unit Set C

This granularity must be preserved during ingestion.

Distributor-level metrics should only be created after the appropriate
aggregation rules have been understood and validated.

---

## 8. Source Data Shape

Initial inspection indicates that continuity indicators are represented in
a long-format structure.

Conceptually:

| Distributor | Consumer Set | Year | Period | Indicator | Value |
|---|---|---:|---:|---|---:|
| Distributor A | Set 01 | 2026 | 1 | DEC | 3.25 |
| Distributor A | Set 01 | 2026 | 1 | FEC | 1.82 |
| Distributor A | Set 01 | 2026 | 2 | DEC | 2.91 |
| Distributor A | Set 01 | 2026 | 2 | FEC | 1.44 |

This structure provides flexibility for storing multiple continuity
indicators but may require transformations for analytical consumption.

The exact structure will be confirmed during data profiling.

---

## 9. Initial Data Relationships

The prototype requires at least two main source domains:

    Service Quality Indicators
              |
              | common business keys
              |
        Regulatory Limits
              |
              v
      Quality Performance

The exact join keys will not be assumed during discovery.

They will be identified and validated using the source data before the
Silver and Gold models are implemented.

---

## 10. Engineering Observations

The discovery phase identified several considerations for the data
engineering solution.

### Multiple Source Resources

Indicators and regulatory limits are published separately.

The ingestion layer must therefore preserve each source independently
before relationships are created in downstream layers.

### Historical Partitions

Historical indicator data is separated into different time ranges.

The ingestion process must support combining these resources without losing
source lineage.

### Reference Data

Indicator descriptions are available as reference data and should be used
instead of unnecessarily hardcoding business metadata into transformation
logic.

### Source Granularity

The original consumer-unit-set granularity should be preserved in the raw
and cleaned layers.

Aggregations should occur only when required by the analytical product.

### Recurring Updates

Because the source is updated periodically, later versions of GridPulse BR
can evolve from full historical ingestion toward incremental processing.

---

## 11. Prototype Data Scope

The first GridPulse BR prototype will use:

- continuity indicator data;
- regulatory limit data;
- indicator reference data when required.

The following sources are intentionally excluded from the first iteration:

- electricity losses;
- tariff data;
- compensation data;
- predictive datasets;
- external economic indicators.

They may be introduced only when a new business requirement justifies the
additional complexity.

---

## 12. Initial Architecture Decision

The source assessment supports the use of a Medallion Architecture.

    ANEEL Open Data
           |
           v
        Bronze
    Raw source data
           |
           v
        Silver
    Cleaned, typed and
     validated records
           |
           v
         Gold
    Business-oriented
       data products
           |
           v
    Databricks SQL
           |
           v
    Quality Analysis

### Bronze

Bronze will preserve source data with minimal transformation and additional
ingestion metadata.

### Silver

Silver will contain cleaned, typed, standardized and validated records.

Relationships between source entities will be introduced only after their
business keys have been validated.

### Gold

Gold will expose business-oriented datasets designed to answer the
prototype's analytical questions.

---

## 13. Open Questions for Data Profiling

The following questions must be answered before the final physical data
model is designed:

1. What are the exact schemas of the indicator and regulatory-limit files?
2. Which fields uniquely identify an indicator observation?
3. Which keys correctly relate indicators to regulatory limits?
4. Are DEC and FEC available consistently across the historical period?
5. Are there duplicate records?
6. How are null values represented?
7. Are numeric values consistently typed?
8. Does `NumPeriodoIndice` represent calendar months for the required
   indicators?
9. Are distributor identifiers consistent across resources?
10. Do schemas differ between historical partitions?
11. How should regulatory limits be associated with reporting periods?
12. What source metadata should be preserved for lineage?

These questions will be answered through exploratory data profiling in
Databricks.

---

## 14. Decision

No final physical schema or ingestion strategy will be defined during the
discovery phase.

The next step is to load and profile representative ANEEL source data in
Databricks.

The results of that profiling will determine:

- Bronze table design;
- source ingestion strategy;
- schema definitions;
- business keys;
- deduplication requirements;
- Silver transformations;
- relationships between indicators and regulatory limits.

This prevents the architecture from being designed based on assumptions
about the source data.
