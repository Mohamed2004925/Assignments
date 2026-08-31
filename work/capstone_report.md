# Content Refresh Opportunity Scoring from an Anonymized Search Snapshot

- **Author:** Mohamed2004925
- **Lane:** Refresh / Content Opportunity Scoring
- **Date:** 2026-08-31
- **Repository:** [Assignments](https://github.com/Mohamed2004925/Assignments)

## Abstract

This FlyRank case study asks whether observed search, freshness, and engagement signals can help content and SEO reviewers allocate limited review time across a content inventory. I evaluated a transparent rule baseline and logistic-regression model using 30,000 pseudonymized content items from the FlyRank ML Internship starter dataset. On a holdout of eight unseen pseudonymized clients, the model reached precision@20 of 0.850 versus 0.700 for the baseline, with average precision of 0.600 and ROC AUC of 0.589 against a 0.517 proxy-label base rate. The model is therefore a modestly better triage signal, not a reliable prediction of future performance or of the result of an edit. The resulting capped, reason-coded queue helps a reviewer decide what to investigate first while retaining human judgment for every content action.

## 1. Introduction / problem statement

In FlyRank's content-intelligence setting, a reviewer may need to assess a large, changing content inventory but cannot inspect every page at once. The case-study decision is therefore which pseudonymized content items should enter a human review queue first, using observed search and engagement signals. The unit of analysis is one content item, and the output is a ranked queue with reason codes and suggested human-review steps. A wrong call wastes reviewer time or leaves a worthwhile page unreviewed, so the system supports investigation rather than automatic publishing, deletion, redirects, or claims about search-engine behavior.

## 2. Data

The model uses the bundled anonymized starter release: 30,000 rows representing content items across 32 pseudonymized clients, with trailing-90-day snapshot metrics. It uses search opportunity, content freshness, and audience-response fields such as impressions, clicks, CTR, average position, content age, update recency, sessions, engagement, scrolling, search volume, competition, CPC, and AI-traffic share.

The paper does not display client names, domains, page URLs, titles, or raw queries. `content_id` and `client_id` are used only to prevent client overlap between train and test; neither is a model feature. The label-derived `trend_direction` and `trend_pct`, along with recent/previous 30-day performance windows that overlap the decline comparison, are excluded to avoid leakage.

## 3. Methodology

The temporary target is the starter snapshot's current-state `decline_proxy_rule`, not a future post-edit outcome. The transparent baseline scores eligible pages using `log1p(impressions_90d) × (1 + days_since_last_update / 365)` after requiring at least 500 impressions and 180 days since an update. The logistic-regression model uses 16 non-identifier, non-label-derived snapshot fields, with preprocessing that preserves missingness information rather than treating all missing values as zero.

Evaluation uses a fixed seed (42) and a client-grouped split: 24 clients / 22,885 rows for training and eight clients / 7,115 rows for test. This estimates behavior on unseen clients more honestly than a random row split. A leakage audit explicitly excluded label-derived fields, overlapping outcome windows, identifiers, and decision-derived fields; adding the forbidden `trend_pct` produced a suspicious ROC AUC of 0.999, which confirms why it is not retained.

## 4. Results

The held-out test proxy-label base rate was 0.517. The baseline and model were compared on the same eight-client split.

| Method | Precision@10 | Precision@20 | Average precision | ROC AUC |
| --- | ---: | ---: | ---: | ---: |
| Transparent freshness × visibility baseline | 0.800 | 0.700 | — | — |
| Logistic regression | 0.900 | 0.850 | 0.600 | 0.589 |

At 20 reviews, the model identified 17 proxy-positive pages on average, compared with 14 for the baseline. This is an observed 0.150 precision@20 improvement on the grouped holdout, not evidence that a content update will cause recovery. The strongest standardized signals by absolute coefficient were days with impressions, days with sessions, and content age. At a 0.50 classification threshold, the model made 3,103 errors out of 7,115 test pages, so scores require review context.

## 5. Limitations and honest framing

The label is a current-state proxy derived from the snapshot, not an observed future decline and not a measure of refresh success. The 30,000-row starter release is a limited anonymized slice, so results may not generalize to other content inventories, tracking configurations, or time periods. The grouped split reduces client memorization but does not replace a time-aware or prospective evaluation. Scores are associations in this dataset; they do not identify causal drivers, predict a search engine's behavior, estimate revenue, or justify automatic changes.

## 6. Ranked recommendations

1. **Review the capped queue first.** Start with the 50 pages meeting the visibility threshold, with no more than three pages per pseudonymized client; this focuses attention while preventing one inventory from monopolizing review.
2. **Diagnose before editing.** For model-flagged candidates, verify analytics coverage, indexing, current search context, intent, seasonality, and content quality before proposing a change.
3. **Treat position opportunities separately.** High-visibility pages with plausible position opportunities should receive a search-results-page and intent-gap review rather than an automatic content refresh.
4. **Record reviewer decisions and outcomes.** Capture why a page was changed or left alone; a later time-aware evaluation can then test whether the queue helps beyond this proxy.
5. **Pause on data-quality changes.** Revalidate after changes to search, analytics, site, or content-template systems; never automate publishing, deletion, redirects, or regulated claims.

The first 50-page queue represents an estimated 53.5 hours of first-pass human review. Its purpose is to order investigation, not to create an automated action list.

## 7. Reproducibility

The analysis notebooks are available in [`work/notebooks/`](https://github.com/Mohamed2004925/Assignments/tree/main/work/notebooks): the baseline, model, validation audit, action playbook, and this capstone notebook. Use Python with the packages listed in [`requirements.txt`](https://github.com/Mohamed2004925/Assignments/blob/main/requirements.txt), then run the notebooks in order from `w01_research_question.ipynb` through `w07_action_playbook.ipynb` and `capstone.ipynb`. The model uses seed 42. Generated metric receipts are retained under `work/outputs/`; datasets and ranked CSV outputs are deliberately not committed.

## 8. Acknowledgments and data credit

Built on the [FlyRank ML Internship dataset](https://flyrank.ai). The dataset is anonymized and used under the project's data-use rules.
