##  Methodology

### 1. Preprocessing & Feature Engineering
Before modeling, the raw data (~36M rows per month) was cleaned and optimized.

- **Ingestion:** Loaded raw TSV data into Spark DataFrames.
- **Cleaning:** Filtered irrelevant 'other-other' records.
- **Optimization:** Saved cleaned data to **Parquet** for faster, columnar read access.
- **Feature Engineering:**
    - **`normalized_clicks`**: The raw click count (`n`) had an extreme range (10 to 167M+), which would break statistical models. We applied a `log1p` transformation to compress this range and make the distribution more normal.
    - **`type_encoded`**: Converted categorical `type` ('link', 'external') to a numerical format for ML models.

### 2. The Three-Model Detection Pipeline

####  Model 1: Statistical (Z-Score)
- **Purpose:** To detect **statistical outliers**—individual navigation paths with an extreme traffic volume. This is designed to find "click bots" or sudden, massive traffic spikes.
- **Why this method?**
    - The **Z-score** ($z = (x-\mu)/\sigma$) is a standard, lightweight method for measuring how many standard deviations a data point is from its mean. It instantly tells us how unusual a single path's traffic is.
    - We used the **3-sigma rule** as our filter. In a normal distribution, 99.7% of all data falls within 3 standard deviations. By filtering for `abs(zscore) > 3`, we isolated the top 0.3% of paths that are statistically exceptional and highly unlikely to be normal human traffic.
- **Implementation:**
    1.  **[Train]** Calculated the `mean` (3.656) and `stddev` (1.198) of `normalized_clicks` from the **Month 1 (July)** baseline.
    2.  **[Test]** Applied these baseline stats to calculate a Z-score for every path in the **Month 2 (August)** test set.
    3.  **[Result]** Flagged **119,506** paths as statistical anomalies.

####  Model 2: Topological (New Paths)
- **Purpose:** To detect **structural anomalies** in the navigation graph. This finds "crawlers" or new behaviors that create previously non-existent paths.
- **Why this method?**
    - A path can be anomalous even if its volume is low. If a path like `Obscure_Article` -> `Main_Page` *never existed before*, its sudden appearance is an anomaly, regardless of volume.
    - This model is volume-agnostic and focuses purely on the *existence* of a path.
- **Implementation:**
    1.  **[Train]** Created a `baseline_edges` set of all distinct `(prev, curr)` pairs from the **Month 1 (July)** data. This is our "map" of all known paths.
    2.  **[Test]** Used a **`left_anti` join** to find all `(prev, curr)` pairs in the **Month 2 (August)** test set that *did not* exist in `baseline_edges`.
    3.  **[Result]** Flagged **1,141,450** new, previously unseen paths.

####  Model 3: Behavioral (K-Means Clustering)
- **Purpose:** To detect **behavioral anomalies**—articles whose *overall traffic profile* is unusual.
- **Why this method?**
    - An article can be anomalous even if no single path to it is a 7-sigma outlier or brand new. Its *pattern* of traffic might change.
    - We create a 7-feature **"traffic profile"** (`in_degree`, `out_degree`, `in_clicks`, `out_clicks`, `ratio_out_in`, `bounce_rate`, etc.) for each article.
    - **K-Means clustering** is an unsupervised ML algorithm that groups articles with similar profiles into "normal" clusters (e.g., a "hub" cluster, a "dead-end" cluster).
    - An article is a behavioral anomaly if its profile is very far (high Euclidean distance) from the center of *any* "normal" cluster.
- **Implementation:**
    1.  **[Train]** Aggregated Month 1 data to build traffic profiles, then trained a `StandardScaler` and `KMeans` (k=10) model to find the 10 "normal" cluster centers.
    2.  **[Test]** Built profiles for Month 2 articles and used the *saved* Month 1 models to assign them to their nearest cluster.
    3.  **[Result]** Calculated the `distance_to_center` for each article and flagged the **top 10** as behavioral anomalies.

---

## Key Findings & Synthesis

The true power of the analysis came from synthesizing the results of all three models.

- **High-Confidence Anomalies:** **5 articles** were flagged by **all three models**.
- **Interpretation:** We successfully distinguished between:
    - **"Expected" Anomalies:** The `Main_Page` was the #1 anomaly. This validates the models, as the Main Page's profile *should* be unique.
    - **"Suspicious" Anomalies:** Article `A` showed a massive Z-score (**7.18**) and a high behavioral distance score, strongly indicating targeted, bot-like activity (a "click bot").

