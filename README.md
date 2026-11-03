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

### Customer conversion model

Three classifiers were compared on the same Spark ML pipeline, then the Random Forest was tuned
with cross-validation:

| Model | Accuracy |
|---|:---:|
| Decision Tree | 0.869 |
| Random Forest | 0.866 |
| Logistic Regression | 0.865 |

**Tuned Random Forest** (100 trees, max depth 8) — the model taken forward:

| Metric | Value |
|---|:---:|
| Accuracy | 0.868 |
| AUC | 0.873 |
| F1 | 0.856 |
| Precision (overall) | 0.885 |

**The headline accuracy hides the part that matters.** Broken out by class, the model identifies
Subscribers reliably but is much weaker on casual Customers — the class the business
wants to find:

| Casual Customer class | Value |
|---|:---:|
| Precision | 0.979 |
| **Recall** | **0.569** |
| F1 | 0.719 |

In other words, when the model flags a casual rider it is almost always right, but it misses
roughly 43% of them. For a conversion campaign that is a real constraint: the targeted list will
be clean but incomplete. Subscribers heavily outnumber casual riders in the data, which is what
drives both the high overall accuracy and the low minority-class recall.

The strongest predictors are start hour (importance 0.34) and gender (0.15) — subscribers ride
at the commuting peaks, casual riders in the middle of the day. Trip duration was deliberately
left out of the features: it is only known once a trip ends, so using it would leak information
the model would not have at the start of a trip. Rider age contributed almost nothing, possibly
because it is missing for many casual riders and was filled with the median.

### Station network & rebalancing

PageRank over the station-to-station trip graph concentrates importance in a small set of
lakefront and downtown hubs — Lake Shore Dr & Monroe St, Streeter Dr & Grand Ave, Lake Shore Dr
& North Blvd, Millennium Park and Michigan Ave & Oak St lead the ranking. Combined with net-flow
(arrivals minus departures), these identify where rebalancing vans have the greatest effect per
stop.

---

## Business Recommendations

1. **Route rebalancing around the top PageRank hubs**, not around raw trip counts. PageRank
   weights a station by the importance of the stations it connects to, which better reflects
   knock-on availability effects across the network.
2. **Use the conversion model for targeting, not for sizing the opportunity.** With 57% recall
   it produces a high-precision shortlist; it will understate how many casual riders are
   convertible.
3. **Improve minority-class recall before running a campaign** — class weighting or threshold
   tuning — if the goal is reach rather than precision.
4. **Time conversion offers around when people ride.** Start hour is the strongest behavioural
   separator between casual and subscribed riders.

---

## Limitations

- **Q3 2019 only** — the peak summer season. Rebalancing and conversion patterns will differ in
  winter, and the findings should not be generalised across the year.
- **Class imbalance was not corrected**, which is why overall accuracy looks strong while
  casual-rider recall does not.
- **The graph analysis treats the season as static**, ignoring time-of-day flow reversal
  (commuter inbound in the morning, outbound in the evening) that matters for real rebalancing.
- Rider age is derived from birth year, which is self-reported and missing for many casual riders.
