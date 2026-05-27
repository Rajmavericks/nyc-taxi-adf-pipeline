# NYC Taxi ADF Pipeline

End-to-end data engineering project built using Azure Data Factory only — no Databricks, no code. Pure visual pipeline processing 6 million NYC Yellow Taxi trip records.

## Tech Stack

| Component | Technology |
|---|---|
| Orchestration | Azure Data Factory (ADF) |
| Transformation | ADF Data Flows (visual, no-code) |
| Storage | Azure Data Lake Storage Gen2 |
| Source Data | NYC TLC Yellow Taxi Trip Records |
| Dashboard | Power BI |

## Architecture

```
NYC TLC Parquet Files (Jan + Feb 2024)
          ↓
ADLS Gen2 — nyctaxi/yellowtaxi/raw/
          ↓  ADF Copy Activity
ADLS Gen2 — nyctaxi/yellowtaxi/processed/
          ↓  ADF Data Flow
            ├── Filter  (remove nulls, bad rows)
            ├── Derived Column  (pickup_date, pickup_hour, duration, revenue_per_mile)
            ├── Aggregate  (group by zone + hour)
            └── Sink
          ↓
ADLS Gen2 — nyctaxi/yellowtaxi/aggregated/
          ↓
Power BI Dashboard
```

## Folder Structure

```
nyc-taxi-adf-pipeline/
  ├── adf/
  │   ├── adfdataset/          ← Dataset JSONs
  │   ├── adflinkedService/    ← Linked Service JSONs
  │   └── adfpipeline/         ← Pipeline JSONs
  ├── datasample/              ← Sample rows for reference
  ├── docs/                    ← Architecture diagram, Power BI file
  └── README.md
```

## Pipelines

### PL_NycTaxi_01_CopyRaw
- Copies raw Parquet files from `raw/` to `processed/` using ADF Copy Activity
- Wildcard source pattern: `*.parquet`
- Preserves file hierarchy

### DF_NycTaxi_Cleanse_Aggregate
ADF Data Flow with 5 transformations:

| Transformation | Description |
|---|---|
| SourceNycTaxi | Reads Parquet from processed zone |
| FilterNulls | Removes rows where fare or distance is null/zero |
| DeriveColumns | Creates pickup_date, pickup_hour, trip_duration_mins, revenue_per_mile |
| AggregateByZoneHour | Groups by date + hour + pickup zone + dropoff zone |
| SinkAggregated | Writes single Parquet to aggregated zone |

### PL_NycTaxi_Master
- Orchestrates Copy + Data Flow in sequence
- Schedule trigger: daily at 6:00 AM IST

## Dataset

Source: [NYC TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

| Property | Value |
|---|---|
| Dataset | Yellow Taxi Trip Records |
| Period | January 2024 + February 2024 |
| Raw size | ~116 MB (2 files) |
| Row count | ~6 million trips |
| Output size | 25.51 MB (aggregated) |

## Key Columns

| Column | Description |
|---|---|
| pickup_date | Trip date (derived) |
| pickup_hour | Hour of day 0–23 (derived) |
| PULocationID | Pickup taxi zone |
| DOLocationID | Dropoff taxi zone |
| total_trips | Trip count per zone-hour group |
| avg_fare | Average fare amount |
| total_revenue | Total revenue per group |
| avg_duration_mins | Average trip duration in minutes |

## Dashboard Insights

- **Peak hour**: 6 PM (hour 18) with 400K+ trips
- **Top revenue zone**: Zone 132 (JFK Airport) — $20M across Jan/Feb
- **Daily revenue**: Consistent ~$5M per day
- **Leap year effect**: Feb 2024 visible drop at day 29

## How to Run

1. Clone this repo
2. Upload source Parquet files to `nyctaxi/yellowtaxi/raw/` in your ADLS container
3. Configure ADF Linked Service to point to your ADLS account
4. Publish all ADF objects from the `adf/` folder
5. Run `PL_NycTaxi_Master` pipeline
6. Connect Power BI to `nyctaxi/yellowtaxi/aggregated/` for dashboard

## Skills Demonstrated

- ADF Linked Services and Datasets
- Copy Activity with wildcard file path
- Data Flows — Filter, Derived Column, Aggregate, Sink
- Master pipeline orchestration with Execute Pipeline activity
- Schedule triggers
- ADLS Gen2 folder structure design
- No-code visual transformations (compiles to Spark under the hood)
- Real-world troubleshooting — compression codecs, schema type resolution


