# Earthquake Analysis: Trends, Prediction, and Clustering

This project analyzes real-time global earthquake data, implementing trend analysis, regression, and clustering to understand where and how strong earthquakes occur.

## Overview

Using the USGS GeoJSON feed, which includes all recorded earthquakes in the past 30 days, it gathers live earthquake data and analyzes it in three stages:

1. **Trend analysis**: frequency over time, magnitude distribution, and feature correlations.
2. **Regression**: predicting earthquake magnitude from location and depth and comparing three different models.
3. **Clustering**: grouping earthquakes geographically to identify high-activity regions and providing a visualization on an actual world map.

## Data source

[USGS Earthquake GeoJSON Feed](https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_month.geojson) - this consists of all earthquakes recorded globally in the past 30 days, using the USGS API. Since this is live data, the results reflect whatever the past 30 days looked like at the time the notebook was run, instead of a fixed time period.

## A bug worth mentioning

GeoJSON stores coordinates in **[longitude, latitude, depth]** order, which is the opposite of what is intuitive. A version of this project stored them in the wrong order, leading to every latitude and longitude value being swapped. Even though it didn't affect the regression models' accuracy (since they treat both coordinates as interchangeable features), it broke every geographic interpretation. This includes the correlation analysis, the cluster map, and the Basemap overlay, which were all technically mislabeled. After noticing this, fixing it and altering the map projection call, which was correct due to the original bug, highlighted why data correctness must be initially verified at the source.

## Key findings

**Trends:** Earthquake frequency surges in periodic spikes rather than occurring at a constant rate, and the majority of recorded earthquakes fall between magnitude 1 and 2, which is consistent with the well-known fact that small earthquakes are far more common than large ones.

**Correlation:** Magnitude has a stronger positive correlation with longitude (0.48), and a stronger negative correlation with latitude (-0.43), highlighting the uneven global distribution of seismic activity.

**Regression (predicting magnitude from latitude, longitude, and depth):**

| Model | R^2 | Approx. Error |
|---|---|---|
| Linear Regression | 0.37 | 58% |
| Decision Tree Regressor | 0.72 | 39% |
| Random Forest Regressor | **0.83** | **30%** |

The tree models outperforming the standard linear regression show that the relationship between location/depth and magnitude is non-linear. A simple linear model underfits, while implementing many decision trees (Random Forest) imitates the pattern far better.

**Clustering:** KMeans (k=4, chosen via the elbow method) identifies distinct earthquake-prone regions that align with well-known seismic zones, such as western North America/Alaska and the western Pacific, which are both segments of the Pacific "Ring of Fire." Overlaying the clusters on an actual world map (via Basemap) makes this pattern immediately visible.

## What this project demonstrates

- Working with a live external API and transforming raw JSON into a structured dataset
- Exploratory data analysis: time series, distributions, and correlation analysis
- Comparing multiple regression models (linear, decision tree, random forest) and reasoning about *why* performance differs between them, rather than just reporting the best score
- Unsupervised learning (KMeans) with a principled method (elbow curve) for choosing cluster count
- Geographic data visualization
- Debugging a data transfer issue (coordinate order) and understanding its downstream effects

## Project structure

```
EarthquakeAnalysis.ipynb   # Full notebook: data pull, trends, regression, clustering
earthquake_data.csv        # Generated when the notebook is run (not committed — see note below)
```

## Running it

```bash
pip install requests pandas matplotlib seaborn numpy scikit-learn basemap
```

Run all cells top to bottom. The notebook pulls live data on each run, so results (frequency spikes, exact R^2 values, cluster locations) will vary slightly depending on when it's executed. This is expected and noted throughout the analysis rather than treated as a fixed benchmark.

## Possible extensions

- Save a timestamped CSV snapshot on each run so it can run historical comparisons
- Add seismic magnitude scale context (e.g. Richter scale energy differences) to the write-up
- Try a gradient-boosted model (XGBoost/LightGBM) as a further comparison point against Random Forest
- Weight the regression loss by magnitude, since large/rare earthquakes are arguably the most important to predict accurately
