# Capstone Report — < Refresh / Content Opportunity Scoring

- **Author: Nioshi Srivastava**
- **Lane: Refresh / Content Opportunity Scoring**
- **Repo: https://github.com/nioshis-cmd/flyrank-ml**
- **Date: 2026-08-25**

## 0. Abstract

This project asks whether safe search-performance signals can be used to prioritize content pages for human review. The analysis uses the FlyRank internship warehouse, with March 2026 selected as the initial development window and one row representing one content item for one client on one day. The baseline prioritizes pages with relatively high search visibility but weaker average search position, and the later modeling work uses a tree-based approach with a client-holdout evaluation design. The observed results are used as a directional comparison between the transparent baseline and the learned model, not as evidence of causation or Google's ranking algorithm. The final output is a ranked content-review queue with reason codes that can help an editor decide which pages deserve attention first.

## 1. Problem framing

The decision supported by this work is: which content pages should a content or SEO team review first?

The unit of analysis is one content item for one client for one day in the selected warehouse data. The output is a ranked opportunity score and action/reason code.

The practical human action is to inspect higher-ranked pages first and decide whether a refresh, improvement, or continued monitoring is appropriate.

A wrong call can waste editorial time or cause a useful page to be changed unnecessarily. Therefore, the score is intended as decision-support, not an automatic action.

Data and ML help because the warehouse contains a large number of page-day observations. A repeatable scoring method can reduce the amount of manual sorting needed and make the prioritization rule consistent.

## 2. Data safety

The analysis uses the FlyRank internship warehouse and the fact_content_daily_performance table. The initial development window used in the data contract is March 1, 2026 through March 31, 2026.

The March window contains 9,841,378 rows, and the grain check found the same number of unique (report_date, client_hash_id, content_hash_id) combinations, supporting the stated one-row-per-client-content-day grain.

The available fields include search-performance signals such as impressions, clicks, average position, pageviews, sessions, engagement measures, and data-availability indicators.

The analysis deliberately excludes or treats the following as non-feature information:

client_hash_id and content_hash_id are pseudonymous identifiers and are used for grouping/evaluation rather than as predictive features.

trend_direction and trend_pct are excluded from predictive features because they are label-derived or outcome-related.

Product decision flags and other pre-existing decision outputs are excluded so the analysis does not simply reproduce an existing product decision.

Raw URLs, queries, titles, client names, domains, credentials, and other identifying/private information are not included in the public report.

A data-availability check found 413,966 rows with GA4 data available out of the 9,841,378 March rows. This is important because GSC and GA4 coverage are not identical across the warehouse.
## 3. Baseline

The baseline is a transparent, human-readable prioritization rule.

It prioritizes pages that have relatively high gsc_impressions but weaker gsc_avg_position. The reasoning is that a page with meaningful search visibility but a weaker average position may be worth review because it already receives visibility while having room for further investigation.

The main reason code is:

HIGH_VOLUME_WEAK_POSITION

Pages that do not match the opportunity pattern receive:

LOW_PRIORITY

This is a fair baseline because the rule is simple, explainable, repeatable, and uses only observable search-performance signals available at the decision point. It provides a clear comparison against the learned model rather than hiding the decision logic inside a complex system.



## 4. Model / analysis

The modeling lane uses a tree-based machine-learning approach to learn relationships between search-performance signals rather than relying only on one hand-written threshold.

The Week-5 notebook describes a Random Forest as the method choice, while its comparison section refers to a Decision Tree. Before the final paper is published, the notebook and this report should use the same final model name. The important methodological point is that the model is tree-based and is being used for ranking/decision-support rather than causal inference.

The candidate feature set is based on safe, observed performance signals, including:

gsc_impressions

gsc_clicks

CTR derived from search impressions/clicks where used

gsc_avg_position

GA4 sessions/pageviews where available

engagement-related measures where used

content age where available

The model does not use client or content identifiers as predictive features, product decision flags, future-window measurements, or label-derived fields.

The target/proxy is an observed content-performance outcome used for prioritization. Any target-derived fields are kept out of the feature set to avoid leakage.
## 5. Evaluation

The primary evaluation design is a client-holdout split. Pages from the same client should not appear in both training and test data, reducing the risk that the model benefits from seeing the same client's patterns in both sets.

The main ranking metric is Precision@50, because the practical use case is to put useful review candidates near the top of a limited human-review queue.

The model and baseline must be evaluated on the same held-out clients and the same metric.
The difference between the two methods should be described as an observed evaluation result. It should not be described as proof that the model will always outperform the baseline.
Error analysis

The model can make both false-positive and false-negative prioritization decisions. A high-scoring page may not actually need a content change, while a genuinely useful review candidate may receive a lower score.

Possible reasons include incomplete search signals, uneven GA4 availability, client-specific differences, and factors about content quality or search intent that are not represented in the warehouse.

For this reason, the ranked queue is a human-review queue, not an automatic publishing or editing system.

## 6. Interpretation

The baseline's main interpretation is straightforward: pages with stronger visibility and weaker average position are treated as potentially useful review candidates.

The learned model is intended to capture more complex combinations of available signals than the baseline's single pattern. The model score therefore represents a ranking signal, not a statement that one feature caused an outcome.

The most important practical interpretation is that search-performance signals can be used to make a large review set easier to prioritize, but the score should still be checked by a human.

Important negative result / caution

The data does not by itself establish why a page performed as it did. It also does not establish that changing a page will cause a particular search outcome.

Any apparent association should therefore be described as observed and directional.
## 7. Recommendation

The recommended workflow is:

Start with the highest-ranked pages in the model queue.

Check the page's recent search performance.

Check whether the search intent still matches the content.

Review content quality and freshness.

Consider business context before making any change.

Choose an action only after human review.

Monitor the result rather than assuming the recommendation was correct.
The confidence level is directional rather than causal. The system is useful for prioritization, but the final editorial decision remains with a human.

The model should never automatically rewrite, delete, merge, publish, or materially change content.

## 8. Reproducibility

The work is organized in the repository under work/notebooks/.

The analysis workflow is:

Define the research question and lane.

Define the data contract and grain.

Check feature leakage.

Audit the selected signals.

Build the transparent baseline.

Train and compare the model.

Re-run the model under an honest client-holdout validation design.

Audit leakage and rewrite claims in safe language.

Produce the ranked action playbook.

Use the resulting outputs to build the capstone paper.

The warehouse is queried through DuckDB over the Hugging Face-hosted release rather than committing raw warehouse data to the repository.

The notebook uses a Hugging Face read token through Colab/user secrets. Credentials are not hard-coded into the repository.

For a fresh run, the notebooks should be executed top-to-bottom in order. The final metrics in this report must match a fresh notebook run.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset.

Data source: https://flyrank.ai

---
Claims checklist

Results are described as observed, measured, directional, or decision-support.

No causal claims are made.

The work does not claim to predict Google's ranking algorithm.

No client-identifying information is included.

Pseudonymous IDs are not used as predictive features.

Label-derived fields and future-window measurements are excluded from the feature set.

Model and baseline are compared on the same evaluation split and metric.

Exact evaluation numbers must be copied from the executed validation notebook before publication.
