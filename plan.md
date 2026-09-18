# HDB Resale Flat Clustering & Forecasting — Discussion Log

## How This Came Up
This was proposed specifically to fill a structural gap in the original brief: a mix of **supervised and unsupervised** learning, across **structured and unstructured** data, demonstrable on both the DA and DS resume versions. COE (chosen first, see its own discussion log) covered supervised + structured. HDB was proposed as the natural pairing to cover **unsupervised + structured** — same real-data, real-community playbook as COE and NTUC, just a different technique.

## Original Concept
Unsupervised segmentation of HDB towns and flat-types by price behavior, so a user could find a town that's "similar to the one I'm looking at, but cheaper" — e.g., "which towns behave like Punggol did 5 years ago." Distribution plan mirrors COE and NTUC exactly: r/singaporefi and r/askSingapore are the real, existing communities where HDB pricing is a constant, high-engagement topic.

## Extension to Price Forecasting
The person later asked directly: "can it also predict what the future sale prices occurring in that area?" This was accepted, but with an explicit caution attached rather than treated as a simple addition. Plain "predict HDB resale prices" is one of the most common Kaggle/tutorial exercises that exists — on its own, it would risk becoming exactly the kind of generic, non-differentiated project the whole portfolio strategy has been trying to avoid (the same concern that ruled out StrataScratch's pre-packaged exercises and flagged the semiconductor idea as risky).

The resolution: the price-prediction component is not the differentiator by itself. The actual contribution is **testing and honestly reporting whether the unsupervised clustering step improves prediction accuracy** — training the same regression/forecasting model twice, once with the cluster assignment included as a feature and once without, and comparing accuracy (MAE/RMSE) between the two. This was explicitly noted as a legitimate finding either way: if clustering *doesn't* meaningfully help, that's still a real, reportable result, not a failed experiment. This comparison is what should be protected under any time pressure — described in the build spec as "if time runs short, protect this comparison over adding more forecasting features."

## Clustering Methodology (locked in before coding)
- **Cluster at the (town, flat-type) aggregation level, not per individual transaction.** Clustering every single sale would mostly just rediscover "expensive sales vs. cheap sales," which isn't a useful or interesting grouping. Aggregating first — one row per town + flat-type combination, with features like median price, price growth rate, and remaining lease — is what actually produces meaningful "neighborhood A behaves like neighborhood B" groups.
- **Choosing k (number of clusters) must not be arbitrary.** The plan is to test a range (e.g., k = 3 through 10) and select via silhouette score, then — critically — sanity-check that the statistically-best k actually produces groupings that make intuitive real-world sense (e.g., do "mature estate" towns actually end up grouped together?). If the statistically optimal k produces nonsensical groupings, that mismatch itself is worth flagging rather than accepting blindly.

## Data Source
`data.gov.sg`'s HDB Resale Flat Prices dataset. **Important open gap:** unlike COE's two datasets, this one's exact current dataset ID/name was never independently verified during scoping — it needs to be looked up directly on the data.gov.sg portal ("HDB resale flat prices") before building starts, since dataset IDs can change over time.

## Tool Choice: Tableau Public (and a correction along the way)
Tableau was deliberately assigned to this project to close a real gap: **Tableau was listed on the resume's Skills section without any project ever demonstrating it** — the same category of issue that led to removing R and Power BI from the skills list earlier in the process, once it was confirmed those tools had genuinely never been used. Rather than removing Tableau too, the decision was to build real, honest experience with it via this project.

Worth noting for anyone picking this file up fresh: an earlier draft of the project's build spec briefly said "Build a Streamlit tool" in one of the approach steps — a leftover from before the Tableau decision was finalized. That inconsistency was caught and corrected; the current spec file correctly says Tableau Public throughout.

## File Format
Pure `.ipynb` end-to-end — clustering, forecasting, and the final CSV export can all live in the notebook. Unlike COE or the running-injury NLP project, this one has **no Streamlit component and therefore no requirement to produce a `.py` file** — Tableau simply consumes a precomputed CSV export from the notebook's final cells.

## Deadline Context (Career Fair, Sept 24, 2026)
HDB was part of an initial "build 3 projects" plan (COE + HDB + NTUC v2) proposed once a hard deadline emerged. Once realistic hourly capacity was calculated (~25–30 hours total over ~19 days at 1–2 hrs/day), the recommendation was walked back to committing fully to COE alone, with HDB explicitly named as a plausible **stretch goal only if COE ships with real time to spare** — not a parallel commitment. HDB was later included again when the person asked to build all three projects with heavier direct assistance (temporarily suspending the default Socratic-teaching build method for this deadline-driven push, to be resumed afterward).

## Coding Standard Note
Same adapted standard as the other projects: since this is Python-based, only the language-agnostic parts of the `/simplify` spec apply (preserve functionality, reduce unnecessary nesting/complexity, avoid nested ternaries or dense one-liners, prefer clarity over cleverness, don't over-consolidate concerns, only refine recently-touched code). The original spec's JS/TS/React-specific rules do not apply here.

## Outstanding / Not Yet Resolved
- **The exact HDB dataset ID/name on data.gov.sg has not been verified** — this is the single most concrete unresolved item for this project and should be the first thing checked before any code is written.
- The exact feature set for clustering (beyond median price, price growth rate, remaining lease) is not finalized — additional features (e.g., proximity to MRT, if available) were mentioned as possibilities but not committed to.
- What "similar" should mean to an end user of the tool (i.e., which features should visibly drive the "find a similar but cheaper town" recommendation) is meant to emerge from the actual clustering/silhouette results, not be assumed upfront.