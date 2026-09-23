# Capstone Report — Machine Learning

- **Author:** Muhammad Arslan Iqbal
- **Lane:** Machine Learning
- **Repo:** https://github.com/arslaniqbalwah/flyrank-ml-internship-arslan
- **Date:** September 24, 2026

## 0. Abstract

Content teams face a massive backlog of pages and struggle to identify which are actively decaying and losing traffic. We analyzed 90-day historical traffic snapshots and content age to model observed decay patterns. Using a Random Forest classifier evaluated on a strict client-grouped split, we created a directional decision-support score for page decline. The model achieved a precision of 70% in the top 100 review queue, significantly outperforming a baseline heuristic of 42%. The final output is a ranked daily action playbook that helps SEO editors prioritize their review workflows effectively.

## 1. Problem framing

This project supports the decision of prioritizing existing content for SEO refreshes. The unit of analysis is a single web page. The output is a directional decision-support score mapped to a ranked daily review queue. A human editor uses this ranked queue to manually verify live search intent before updating the page. The cost of a wrong call is wasted editorial hours on healthy pages, or lost organic traffic from ignoring genuinely decaying pages. Data and ML are necessary here because the relationship between traffic volume, age, and decay is non-linear, making simple static heuristic rules blunt and error-prone.

## 2. Data safety

I used the `content_refresh_anonymized.csv` dataset. I explicitly excluded `client_id`, `content_id`, and any URL strings from the feature set to prevent the model from memorizing specific client patterns or domain authority. The target label was derived solely from `trend_direction`, so I ensured no other trend or future-looking fields (like `trend_pct`) were used as inputs to prevent data leakage. `client_id` was strictly reserved for creating a leakage-free grouped evaluation split. I confirm no client-identifying details appear anywhere in the `work/` directory.

## 3. Baseline

The baseline was a transparent, hand-crafted rule prioritizing high-traffic pages with a bonus weight for newer content: `score = impressions_90d (x1.5 if content_age_days < 365)`. This is a fair comparison because it represents a standard industry heuristic ("update high-impact pages before they get too old"). Evaluated on the exact same grouped split and metric (Precision@100), the baseline achieved 42.00%.

## 4. Model / analysis

I chose a Random Forest Classifier (with `max_depth=5` to prevent overfitting) because it effectively handles non-linear interactions between age and traffic without requiring heavy scaling or normalization. The exact feature list used was: `impressions_90d`, `sessions_90d`, and `content_age_days`. The proxy target was defined as `is_declining`, which equates to `trend_direction == 'down'`.

## 5. Evaluation

I utilized a `GroupShuffleSplit` on `client_id` (`test_size=0.2`). This was critical; a random split would allow the model to cheat by memorizing a specific client's traffic footprint across training and test sets. On this honest, unseen-client split, the ML model achieved a Precision@100 of 70.00%, compared to the baseline's 42.00%. A short error analysis reveals the model still produces false positives on niche pages—it occasionally misinterprets naturally low, stable traffic volume as a measured signal of decay. 

## 6. Interpretation

The model relies heavily on historical traffic volume to make its splits. Feature importances measured `impressions_90d` as the strongest signal (0.582), followed by `content_age_days` (0.321), and `sessions_90d` (0.097). The primary surprise was the reversal of a common SEO assumption: our earlier signal audit demonstrated that newer pages in this dataset actually exhibited a higher baseline decay rate than older, stabilized pages, which the tree model successfully internalized.

## 7. Recommendation

The output supports a prioritized workflow where pages are assigned an action (`review_for_refresh`) and a reason code (`high_probability_decay` for ML score > 0.7, or `moderate_decay_risk` for 0.5-0.7). A FlyRank editor should pull the top 100 pages daily and manually review the SERP intent before acting. This model is strictly a directional prioritization engine. It is blind to macro seasonality, competitor actions, and Google algorithm updates, and should never be used for automated unpublishing or redirection.

## 8. Reproducibility

To reproduce this work from a fresh clone:
1. Clone the repository: `git clone https://github.com/arslaniqbalwah/flyrank-ml-internship-arslan.git`
2. Install standard dependencies: `pandas`, `scikit-learn`, `matplotlib`.
3. Run `work/notebooks/capstone.ipynb` top-to-bottom.
The environment relies on a standard Python 3 setup. Random seeds (`random_state=42`) were fixed in both the `GroupShuffleSplit` and the `RandomForestClassifier` to ensure deterministic, strictly sealed metrics. The CSV output and charts are systematically generated into the `work/outputs/` directory.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset, provided by [https://flyrank.ai](https://flyrank.ai).
