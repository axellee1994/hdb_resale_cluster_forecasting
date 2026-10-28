# HDB Resale Clusters and Price Forecasting

**Live dashboards: [axellee1994.github.io/hdb_resale_cluster_forecasting](https://axellee1994.github.io/hdb_resale_cluster_forecasting/)** (two Tableau Public views)

Groups Singapore's HDB resale market into clusters of (town, flat type) combinations that behave alike, so a buyer can find towns that resemble the one they are looking at but at a different price. Then it tests one question: **does knowing the cluster make a price model more accurate?**

## The result

The same gradient-boosting model was trained twice, with and without the cluster label, and scored on the last 6 months of sales (13,657 transactions, May to Oct 2026). Neither the model nor the clusters saw those months.

| Holdout (May-Oct 2026) | MAE | RMSE | MAPE |
|---|--:|--:|--:|
| Without cluster | $46,293 | $67,305 | 6.73% |
| With cluster | **$45,479** | **$64,996** | **6.65%** |

Adding the cluster cuts MAE by 1.8% and RMSE by 3.4%. A small but consistent gain: the clusters carry town-level price behaviour that the town and flat-type codes alone miss. An earlier run on data to Sep 2026 gave 3.3% (MAE), so the size of the gain moves with the holdout window.

## The clusters

243,652 resale transactions (Jan 2017 to Oct 2026) are reduced to 126 (town, flat type) combinations with at least 30 sales each. Each is described by median price, price growth and average remaining lease, then grouped with K-Means.

| Cluster | Combinations | Median price | Price growth | Avg remaining lease | Typical members |
|---|--:|--:|--:|--:|---|
| Premium | 43 | $790,000 | 24.4% | 68.7 years | 4-room to executive flats in mature estates |
| Newer, High-Growth | 53 | $458,000 | 39.7% | 79.2 years | Punggol, Sengkang, Woodlands, Sembawang |
| Mature & Affordable | 30 | $355,000 | 31.1% | 59.3 years | 1- to 4-room flats with older leases |

- **k = 3** has the best silhouette score (0.403) of k = 3 to 10, and the groups make real-world sense.
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
- **Clusters built from training months only**, so the cluster label carries no information about holdout prices.
- **One change between the two models.** Same features, same hyperparameters, same split; only the `cluster_id` column differs.
- **Clusters are chosen by the data and then sanity-checked**, not picked by hand.

## Limitations

- **The data changes.** data.gov.sg adds late-registered sales to past months, so a rerun gives slightly different numbers.
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
