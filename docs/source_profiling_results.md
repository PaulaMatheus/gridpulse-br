# Source Profiling Results

## GridPulse BR — ANEEL Continuity Indicators

This document summarizes the results of the initial source profiling
performed on the ANEEL Collective Continuity Indicators dataset.

The profiling was performed before designing the physical Medallion
Architecture to ensure that engineering decisions were based on observed
source behavior rather than assumptions.

---

## 1. Source Overview

| Attribute | Result |
|---|---|
| Provider | ANEEL |
| Dataset | Collective Continuity Indicators |
| Source format | Apache Parquet |
| Historical resource | 2020–2029 |
| Records currently available | 5,042,862 |
| Available years | 2020–2026 |
| Latest available period | July 2026 |
| Distributors | 105 |
| Consumer-unit sets | 3,980 |
| Indicators | 23 |
| Null values | None identified |

The source was retrieved programmatically from the ANEEL Open Data Portal
and stored without modification in the project's landing area.

---

## 2. Source Schema

The source contains the following fields:

| Field | Source Type | Description |
|---|---|---|
| `DatGeracaoConjuntoDados` | date | Dataset generation date |
| `IdeConjUndConsumidoras` | long | Consumer-unit set identifier |
| `DscConjUndConsumidoras` | string | Consumer-unit set description |
| `SigAgente` | string | Electricity distributor code |
| `NumCNPJ` | long | Distributor CNPJ |
| `SigIndicador` | string | Continuity indicator code |
| `AnoIndice` | long | Indicator year |
| `NumPeriodoIndice` | long | Reporting period |
| `VlrIndiceEnviado` | double | Reported indicator value |

A relevant modeling observation is that `NumCNPJ` is represented as a
numeric value in the source.

Because CNPJ is a business identifier rather than a measurable numeric
attribute, its representation will be reviewed during Silver-layer
standardization.

---

## 3. Temporal Coverage

The dataset currently contains the following temporal coverage:

| Year | Available Periods |
|---:|---:|
| 2020 | 12 |
| 2021 | 12 |
| 2022 | 12 |
| 2023 | 12 |
| 2024 | 12 |
| 2025 | 12 |
| 2026 | 7 |

The current year is therefore partially available through July 2026.

This behavior is consistent with a periodically updated operational source
and will be relevant when freshness monitoring and incremental ingestion
are introduced in later iterations.

---

## 4. Dataset Grain

Profiling indicates that an indicator observation is conceptually
identified by:

- consumer-unit set;
- indicator;
- year;
- reporting period.

The initial candidate business key is therefore:

```text
IdeConjUndConsumidoras
+
SigIndicador
+
AnoIndice
+
NumPeriodoIndice
