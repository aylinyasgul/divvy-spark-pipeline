# Divvy Bike Q3 2019 — Spark ML & GraphFrames

Analysis of Chicago's Divvy bike-share network using Apache Spark, covering the peak summer season (July–September 2019). The project answers two business questions through a single end-to-end PySpark pipeline.

## Business Questions

| # | Problem | Approach |
|---|---|---|
| 1 | **Network Imbalance** — which stations accumulate or drain bikes during the season, and how should rebalancing vans be routed? | GraphFrames: PageRank, Degree centrality, Net Flow, Triangle Count |
| 2 | **Customer Conversion** — can we predict whether a rider is a Subscriber or casual Customer from ride behaviour, and identify likely converters to target with discounts? | Spark ML: Random Forest Classifier (benchmarked against Logistic Regression and Decision Tree) |

## Tech Stack

| Layer | Technology |
|---|---|
| Compute | Apache Spark (PySpark) |
| Graph analytics | GraphFrames |
| Storage | HDFS |
| ML | Spark MLlib (Pipeline API) |
| Visualisation | Pandas, Matplotlib |
| Notebook | Jupyter |

## Dataset

[Divvy Bike-Sharing Q3 2019](https://divvybikes.com/system-data) — publicly available from the City of Chicago.  
~1.6 million trips, July–September 2019. The CSV is **not** committed to this repo (215 MB); download it from the link above and place it in HDFS as described below.

## Prerequisites

- Java 8 or 11
- Apache Spark 3.x with PySpark
- GraphFrames package (`graphframes:graphframes:0.8.2-spark3.0-s_2.12` or compatible)
- HDFS running locally or on a cluster
- Python 3.8+ with `pandas` and `matplotlib`

## Setup & Running

### 1. Start HDFS and place the data

```bash
# Put the CSV into HDFS at the path the notebook expects
hdfs dfs -mkdir -p /datalake/raw/divvy/
hdfs dfs -put Divvy_Trips_2019_Q3.csv /datalake/raw/divvy/
```

### 2. Launch Jupyter with the GraphFrames package

```bash
pyspark \
  --packages graphframes:graphframes:0.8.2-spark3.0-s_2.12 \
  --master local[*]
```

Or start Jupyter separately and set `PYSPARK_DRIVER_PYTHON=jupyter` before launching `pyspark`.

### 3. Open the notebook

Open `Divvy_Spark_Pipeline.ipynb` and run all cells in order.  
The notebook is self-contained — each section builds on the previous one.

## Notebook Structure

1. Environment setup & imports
2. Spark session creation
3. Load & inspect data from HDFS
4. Cleaning & feature engineering
5. Exploratory Data Analysis
6. Spark ML — Customer Conversion Classifier
7. GraphFrames — Station Network & Imbalance
8. Visualisations & business insights
9. Summary of findings

## Key Findings

- The Random Forest classifier achieves high accuracy in predicting Subscriber vs Customer status, driven mainly by `trip_duration` and `rider_age`.
- PageRank and net-flow metrics identify a small set of high-traffic hub stations where rebalancing effort has the greatest impact.
