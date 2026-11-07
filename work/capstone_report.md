# Can search signals prioritize content pages for human review?

**Pratheek Rao** · Lane 2 — Refresh / Content Opportunity Scoring · March 2026

<div class="abstract" id="abstract"><strong>Abstract</strong><p>Can early search signals help teams decide which content pages to review first? We analyze March 2026 from the FlyRank internship warehouse build v20260703, using 151,981 early-period candidates and an 85,565-page subset with complete later-period measurements. We compare a logistic-regression ranking over five March 1–15 features with the fixed three-signal Week 4 rule on the same client-grouped holdout, where 31 clients train and 8 clients are held out with no overlap. On 4,113 held-out pages, both rankers have Precision@20 of 0.55; the model reaches 0.48 versus 0.46 Precision@50 and 0.767 versus 0.572 ROC-AUC, while the decline-proxy base rate is 0.322. The result supports using the ranking as a human review aid in this historical sample, not predicting future performance or establishing that a refresh improves outcomes.</p></div>

<div class="callout"><strong>Finding.</strong> On the grouped holdout, the model tied the transparent rule at Precision@20, edged it at Precision@50, and had higher ROC-AUC. This supports human review prioritization in the observed sample; it does not show that refreshing a page improves performance.</div>

## 1. Introduction / problem statement {#problem}

An editor cannot inspect every page at once. This capstone asks whether early search-performance signals can help order a review queue so a person can inspect promising candidates first. The unit is one pseudonymized content page; the output is a rank and a transparent review/monitor suggestion. A false positive costs reviewer time, while an unreviewed page may leave a possible opportunity unnoticed. No intervention was measured, so this is a prioritization study rather than an estimate of refresh impact.

## 2. Data {#data}

The source is the gated FlyRank internship warehouse, build **v20260703**, specifically `fact_content_daily_performance` for March 2026. The full release contains **78,835,655** daily performance rows; the March partition contained **9,841,378** page-day rows, of which **3,611,061** had `gsc_data_available = TRUE`. Those rows are aggregated to one pseudonymized client/page pair.

There are **151,981** early candidates with usable March 1–15 search measures. The retrospective label is available for **85,565** pages with all 16 dates from March 16–31. Another **55,902** pages had only 1–15 later dates and were excluded from label evaluation. Queue construction still covers the full early candidate frame and does not depend on later availability.

The June `_sample` and overlapping 90-day query table were not used. Client/content pseudonyms support grouping, splitting, and deterministic tie-breaking only; they are not features. Later-period values, target-derived trends, product flags, and action labels were excluded from the feature set. No raw queries, page URLs, client names, or row-level exports are published here.

## 3. Methodology {#methods}

Five prediction-time features use March 1–15: early impressions, early clicks, impression-weighted average position, available search-day count, and early CTR (`100 × clicks / impressions`, as a percentage). The decline proxy is 1 when mean daily impressions over March 16–31 are lower than mean daily impressions over March 1–15, provided all 16 later dates are available.

The unchanged Week 4 rule adds one point for at least 500 early impressions, one for a positive weighted position no worse than 20, and one for CTR below 0.5%; only score 3 suggests `review_ctr_refresh`. The learned ranker is a standardized logistic-regression pipeline with class-balanced fitting, `max_iter=1000`, and `random_state=42`.

Evaluation uses `GroupShuffleSplit(test_size=0.20, random_state=42)` by client. It has 81,452 training pages from 31 clients and 4,113 test pages from 8 clients, with zero overlap. The model and baseline use the exact same test rows and labels. The Week 3/6 leakage checks excluded future-period fields, target-derived trends, action/product flags, and IDs from the feature vector; preprocessing was fit on training pages only.

## 4. Results (model versus baseline) {#results}

On the grouped holdout, the model ties the rule at Precision@20, has a two-page advantage in the top 50, and has higher ROC-AUC in this one split. The 32.2% positive base rate is shown beside both methods. The AUC difference describes ranking discrimination on these eight held-out clients, not guaranteed performance on new clients.

| Method | Precision@20 | Precision@50 | ROC-AUC | Base rate |
|---|---:|---:|---:|---:|
| Week 4 rule baseline | 0.55 (11/20) | 0.46 (23/50) | 0.572 | 0.322 |
| Logistic regression | 0.55 (11/20) | 0.48 (24/50) | 0.767 | 0.322 |

![Grouped holdout comparison on the same 4,113 pages. Both methods have Precision@20 of 0.55; Precision@50 is 0.46 for the baseline and 0.48 for logistic regression; ROC-AUC is 0.572 and 0.767. The holdout base rate is 0.322.](assets/heldout_comparison.svg)

**Figure 1.** Both methods use the same grouped test set. Dashed markers in the precision rows show the 0.322 positive-class rate.

The transparent rule flags **26,597** of 151,981 candidates (**17.5%**) for human CTR review; 125,384 remain in monitor status.

![Action queue contains 26,597 pages suggested for CTR review and 125,384 pages to monitor, out of 151,981 early-period candidates.](assets/w07_action_distribution.svg)

**Figure 2.** Week 7 action distribution. A review suggestion is a prompt for human inspection, not an instruction to edit or publish.

Within the evaluable subset, low early CTR (<0.5%) is associated with a higher decline-proxy rate in each measured position band. This is observational and does not show that changing a title, snippet, or page will improve outcomes.

![Across four position bands, the observed decline-proxy rate is higher in the low-CTR group than in the at-least-0.5-percent CTR group.](assets/w07_decay_proxy_by_position.svg)

**Figure 3.** Historical decline-proxy rate by early CTR and position. Rates use the 85,565-page evaluable subset; subgroup sizes and observational design limit interpretation.

## 5. Limitations & honest framing {#limits}

The outcome is a historical within-month proxy, not future decline, search-engine ranking success, or a causal refresh effect. Only 8 clients are in the grouped holdout, and a single seeded split does not capture split-to-split variability. The complete-label subset may differ from pages with incomplete later measurements. Seasonality, competition, indexing, and measurement conditions are not represented by the five features. The action queue and model metrics apply to different populations: the queue uses the Week 4 rule over all early candidates, while model metrics apply to the evaluable grouped holdout. No monetary cost or editor acceptance data was available.

## 6. Ranked recommendations {#recommendations}

1. **Review score-3 pages first.** For pages meeting all three rule conditions, inspect current visibility, search intent, title/snippet, indexing, freshness, competitors, and seasonality. These 26,597 pages are candidates for human review, not proven refresh opportunities.
2. **Keep score-2 pages in monitor status.** The 7,612 visible-demand-near-opportunity pages can be considered when capacity remains, after contextual checks. Do not silently expand the rule based on the model result.
3. **Monitor low-signal pages and improve evidence.** When signals are weak or unavailable, wait for enough data rather than infer that content should be pruned.

Do not automate rewriting, publishing, deletion, redirects, canonical/indexation changes, or high-impact decisions from a score alone. Record accept/defer/reject reasons. Monitor queue composition, human acceptance, later Precision@K, feature drift, base-rate change, and ranking stability; proposed Week 7 thresholds are starting points, not validated operating limits.

## 7. Reproducibility {#reproducibility}

The executed capstone reconstructs the March frame from the gated parquet partition, fits the grouped model, compares the Week 4 rule on the identical holdout, exports aggregate metrics, and regenerates this page's figures. A fresh run requires access to `FlyRank/internship-warehouse` and a locally configured `HF_TOKEN`; the token is read from the environment or ignored `.env` and is never written to notebook output.

```bash
pip install -r requirements.txt
jupyter notebook work/notebooks/capstone.ipynb
```

- [Repository](https://github.com/pratheekrao12/-flyrank-ml-internship)
- [Capstone notebook](https://github.com/pratheekrao12/-flyrank-ml-internship/blob/main/work/notebooks/capstone.ipynb)
- [Week 6 validation audit](https://github.com/pratheekrao12/-flyrank-ml-internship/blob/main/work/notebooks/w06_validation_audit.ipynb)
- [Week 7 action playbook](https://github.com/pratheekrao12/-flyrank-ml-internship/blob/main/work/notebooks/w07_action_playbook.ipynb)

The group split seed is 42. `work/outputs/capstone_metrics.json` records the aggregate results and feature definitions.

## 8. Acknowledgments & data credit {#acknowledgments}

Built on the [FlyRank ML Internship dataset](https://flyrank.ai).
