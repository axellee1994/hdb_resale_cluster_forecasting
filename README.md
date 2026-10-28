# HDB Resale Clusters and Price Forecasting

**Live dashboards: [axellee1994.github.io/hdb_resale_cluster_forecasting](https://axellee1994.github.io/hdb_resale_cluster_forecasting/)** (two Tableau Public views)

Groups Singapore's HDB resale market into clusters of (town, flat type) combinations that behave alike, so a buyer can find towns that resemble the one they are looking at but at a different price. Then it tests one question: **does knowing the cluster make a price model more accurate?**

## The result

The same gradient-boosting model was trained twice, with and without the cluster label, and scored on the last 6 months of sales (12,574 transactions, Apr to Sep 2026). It never saw those months in training.

| Holdout (Apr-Sep 2026) | MAE | RMSE | MAPE |
|---|--:|--:|--:|
| Without cluster | $44,555 | $63,761 | 6.58% |
| With cluster | **$43,067** | **$60,962** | **6.39%** |

Adding the cluster cuts MAE by 3.3% and RMSE by 4.4%. A small but real gain: the clusters carry town-level price behaviour that the town and flat-type codes alone miss.

## The clusters

240,610 resale transactions (Jan 2017 to Sep 2026) are reduced to 125 (town, flat type) combinations with at least 30 sales each. Each is described by median price, price growth and average remaining lease, then grouped with K-Means.

| Cluster | Combinations | Median price | Price growth | Avg remaining lease | Typical members |
|---|--:|--:|--:|--:|---|
| Premium | 42 | $799,500 | 26.8% | 68.8 years | 4-room to executive flats in mature estates |
| Newer, High-Growth | 53 | $462,500 | 41.2% | 79.0 years | Punggol, Sengkang, Woodlands, Sembawang |
| Mature & Affordable | 30 | $358,400 | 32.7% | 59.3 years | 1- to 4-room flats with older leases |

- **k = 3** has the best silhouette score (0.401) of k = 3 to 10, and the groups make real-world sense.
- **Stable:** across 20 runs with different random seeds, every pair of combinations either always or never landed in the same cluster.

## How it works

```
data.gov.sg API (HDB resale flat prices, ~241 pages of 1,000 rows)
        |
        v
hdb_clustering.ipynb
   1. Fetch and cache the raw data       -> dataset/hdb_resale_raw.csv
   2. Clean: parse remaining lease and storey, drop combos with < 30 sales
   3. Explore: prices by flat type, volume over time, town trajectories
   4. Cluster: aggregate to (town, flat type), scale, K-Means k = 3-10, silhouette, stability
   5. Forecast: same model with and without the cluster, temporal holdout
   6. Export                             -> dataset/hdb_tableau_export.csv
        |
        v
Tableau Public dashboards (published at docs/index.html)
```

**Kept honest by construction:**

- **Temporal split, not random.** A random split would let the model train on 2026 sales and "predict" other 2026 sales. The holdout is the last 6 months, fixed before any model is fitted.
- **One change between the two models.** Same features, same hyperparameters, same split; only the `cluster_id` column differs.
- **Clusters are chosen by the data and then sanity-checked**, not picked by hand.

## Limitations

- **The clusters were built on the full period, including the 6 holdout months.** The cluster label therefore carries a little information about holdout prices, which may flatter the with-cluster model. Building the clusters on training months only is the clean fix; until then, treat the 3.3% gain as an upper bound.
- **One holdout window.** A rolling backtest over several 6-month windows would show whether the gain holds across market conditions.
- **No location detail below town level** (distance to MRT, schools); these were considered but not added.

## Running locally

```bash
pip install -r requirements.txt
jupyter notebook hdb_clustering.ipynb    # Restart Kernel -> Run All
```

The first run fetches about 240,000 rows from data.gov.sg (several minutes) and caches them in `dataset/hdb_resale_raw.csv`. Delete that file to force a fresh fetch.

## Repository layout

| Path | What it is |
|---|---|
| `hdb_clustering.ipynb` | The full analysis: fetch, clean, explore, cluster, forecast, export |
| `dataset/` | Cached raw data and the Tableau export |
| `docs/index.html` | GitHub Pages site embedding the two Tableau dashboards |
| `plan.md` | Design notes written before coding |
| `requirements.txt` | pandas, NumPy, scikit-learn, statsmodels, Matplotlib, requests |

## Data

[HDB resale flat prices](https://data.gov.sg/datasets/d_8b84c4ee58e3cfc0ece0d773c8ca6abc/view) (HDB via data.gov.sg), one row per resale transaction from Jan 2017.

Not financial advice.
