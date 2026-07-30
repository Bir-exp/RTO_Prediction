# Results and Findings — RTO Prediction Analysis

## Executive Summary

This analysis developed an RTO (Return to Origin) prediction model. The best-performing model, Gradient Boosting, achieved an **F1-Score of 68.75%**, an **accuracy of 66.32%**, and a **ROC-AUC of 73.33%**. These results are modest rather than strong in absolute terms — accuracy is only slightly better than chance for a roughly balanced problem — but the model does provide useful signal for ranking shipments by RTO risk, and the feature importance results offer directionally useful insight for logistics interventions.

---

## 1. Model Performance Results

### Best Model: Gradient Boosting

| Metric | Value | Notes |
|--------|-------|-------|
| F1-Score | 68.75% | Harmonic mean of precision and recall; the two are not separately reported here and should not be assumed equal to this figure |
| Accuracy | 66.32% | Overall correct-classification rate |
| ROC-AUC | 73.33% | Reasonable, not strong, class discrimination |
| Cross-Validation F1 | 70.66% ± 1.42% | Consistent across folds, suggesting the estimate is stable rather than a lucky split |

### Full Model Comparison

| Rank | Model | F1-Score | Accuracy | ROC-AUC | CV F1 |
|------|-------|----------|----------|---------|-------|
| 1 | Gradient Boosting | 68.75% | 66.32% | 73.33% | 70.66% ± 1.42% |
| 2 | Decision Tree | 68.71% | 66.09% | 72.24% | 69.92% ± 2.66% |
| 3 | Random Forest | 68.43% | 66.36% | 72.06% | 70.14% ± 0.92% |
| 4 | Extra Trees | 66.10% | 65.82% | 71.75% | 67.09% ± 0.21% |
| 5 | Logistic Regression | 56.64% | 64.23% | 72.49% | 58.72% ± 1.68% |

**Observation:** The top four models are close in performance (within ~2.6 points of F1). Gradient Boosting's lead over Decision Tree and Random Forest is small and may not be practically significant. Logistic Regression trails badly on F1 despite a competitive ROC-AUC, which suggests its default decision threshold is poorly calibrated for this problem rather than that it captures less information — worth checking before ruling it out for interpretability reasons.

---

## 2. Feature Importance Analysis

### Top Features (Gradient Boosting)

| Rank | Feature | Importance | Interpretation |
|------|---------|------------|-----------------|
| 1 | Weight_in_gms | 27.51% | Package weight is the single largest driver of predicted RTO risk |
| 2 | Discount_offered | 27.44% | Discount level is nearly as important as weight |
| 3 | Cost_Weight_Ratio | 15.14% | An engineered feature combining cost and weight; adds importance beyond either alone |
| 4 | Weight_Bin | 12.22% | A binned version of weight; note this is likely correlated with Weight_in_gms and their importances should not simply be added together |
| 5 | Cost_of_the_Product | 4.24% | Secondary driver |
| 6 | Prior_purchases | 3.98% | Customer history has a small but non-trivial effect |
| 7 | Distance_Weight_Ratio | 3.49% | Delivery-distance interactions play a minor role |
| 8 | Cost_Bin | 2.71% | Minor |
| 9 | Customer_care_calls | 1.71% | Minor |
| 10 | Customer_rating | 0.40% | Negligible — customer ratings carry almost no predictive weight |

**Caution on interpretation:** Feature importance scores from tree-based models reflect how useful a feature was for splitting during training — they indicate association with the outcome, not proven causation. "Heavier packages have higher RTO risk" is a reasonable hypothesis these numbers support, but the model itself does not establish that reducing package weight would *cause* fewer returns. Also, because Weight_in_gms, Weight_Bin, Cost_Weight_Ratio, and Cost_of_the_Product are all derived from the same one or two underlying variables (weight and cost), their combined ~59% share of importance overstates the number of independent signals in the data — it is closer to two strong underlying factors (weight, cost/discount) than four.

---

## 3. Visualizations Referenced

The original analysis included the following plots (not reproduced here): a model-performance comparison chart, a precision-recall curve, and a feature-importance bar chart / heatmap. Descriptive claims about what these charts "show" (e.g., "steep curves indicate good discrimination") should be read as qualitative impressions rather than quantified findings — no separate precision/recall or curve-shape numbers were provided alongside the summary metrics above.

---

## 4. Business Interpretation

### What the model supports

- The model can rank shipments by estimated RTO risk with meaningfully better accuracy than a coin flip, but with a real error rate: roughly one in three predictions is wrong at 66% accuracy.
- Weight, discount level, and cost are the strongest available signals in this dataset for RTO risk.
- Customer rating and customer-care-call volume are weak predictors in this model, which is a useful negative finding — it suggests these are not good targets for RTO-reduction efforts based on this data.

### What the model does not establish

- **Precision and recall**: only F1 is reported; without the individual values, claims like "69% of predicted RTOs are actual returns" cannot be made from this data as written (F1 is not precision, and conflating them overstates confidence in either metric).
- **Cost savings and ROI**: figures such as "reduce RTO rates by 15–25%" or "ROI within 6 months" are not derived from the model output above. They are illustrative business targets and should be labeled as assumptions to be validated, not as projected outcomes of the model.
- **Causality**: correlations between weight/discount and RTO do not by themselves justify specific interventions (e.g., "special handling for packages > 2000g") — that threshold is not derived from the analysis and would need separate validation.

### Suggested framing for stakeholders

| Intervention Area | Basis in the Data | Confidence |
|---|---|---|
| Review handling/communication for heavy packages | Weight is the top feature by importance | Moderate — association, not causally confirmed |
| Reassess discount policy on high-value items | Discount is the second-largest feature | Moderate — association, not causally confirmed |
| Deprioritize customer-rating-based interventions for RTO specifically | Near-zero importance (0.40%) | Reasonably confident, given model performance caveats |
| Assume specific % reduction in RTO or dollar ROI | Not present in current results | Low — requires a separate cost/impact study or A/B test |

---

## 5. Methodology Notes

- **Model choice**: Gradient Boosting was selected for having the highest F1-Score among five tested models, though its margin over Decision Tree and Random Forest is small.
- **Feature engineering**: Engineered features (Cost_Weight_Ratio, Weight_Bin, Distance_Weight_Ratio) contribute meaningfully to importance rankings, but as noted above, some of this is redundant with their source variables rather than fully independent signal.
- **Evaluation metric**: F1-Score is a reasonable choice for an imbalanced classification problem, since accuracy alone can be misleading when one class is more common. The original document is correct on this point.
- **Cross-validation**: 5-fold CV with a tight standard deviation (±1.42% for the best model) indicates the F1 estimate is stable across data subsets, which is a genuine strength of this analysis.
- **Data**: 10,999 records with no missing values, per the original report.

---

## 6. Revised Recommendations

1. **Report precision and recall separately** before making any claims about "percent of predictions correct" — this is a gap in the current results, not a detail to gloss over.
2. **Treat the 15–25% RTO reduction and 6-month ROI figures as placeholders**, not findings, until they are backed by a cost model or pilot test.
3. **Use the model for risk ranking/triage, not as a standalone decision rule** — at 66% accuracy, any automated action (e.g., withholding shipment) should have a human review step or a conservative threshold.
4. **Investigate collinearity** between Weight_in_gms, Weight_Bin, Cost_Weight_Ratio, and Cost_of_the_Product before making resourcing decisions based on their combined importance.
5. **Deprioritize customer-rating-driven RTO initiatives** given its negligible importance, and consider testing pricing/discount-threshold changes given the strength of that signal.

---

*This revision keeps the original model results as reported but removes claims (precision/recall figures, ROI percentages, causal language) that were not actually supported by the metrics provided, and flags where the original document's confidence exceeded what a 66% accuracy / 68.75% F1 model can reasonably justify.*
