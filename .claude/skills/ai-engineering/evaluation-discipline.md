# Evaluation Discipline

Model evaluation is where most ML projects are dishonest, usually unintentionally. These are the signals and reasoning prompts for catching evaluation problems before they propagate to production.

---

## The train/validation/test split contract

**The contract:** The test set is touched exactly once — after the model is fully developed and hyperparameters are chosen. It measures generalisation.

**Violations:**
- Using the test set to choose between models or tune thresholds (now it is a second validation set, not a test set — you need a new held-out set)
- Choosing the best model by test set performance over multiple iterations
- Reporting test set performance during development to inform next steps

**Signal that the contract is violated:** You have a number that feels like test accuracy but you ran the evaluation multiple times. That number is optimistic.

---

## Metric selection

The choice of metric is a design decision, not a convenience.

**Accuracy is almost always the wrong metric for imbalanced problems.** A model that predicts the majority class for every input achieves high accuracy on an imbalanced dataset. This is not a useful model.

**Ask for each metric:**
- Does this metric capture what failure actually costs? (A false negative in fraud detection costs differently from a false positive.)
- Is this metric gameable? Can the model optimise this metric without solving the actual problem?
- Is this the metric the business cares about, or is it a proxy?

**Common metric traps:**
- AUC is a summary over all thresholds — it can be high while the model is useless at any specific threshold used in production
- BLEU/ROUGE scores for text generation correlate weakly with human judgment
- Aggregated metrics hide subgroup failures — a model with 90% overall accuracy might have 60% accuracy on the minority class

---

## Offline vs. online evaluation

**Offline evaluation** (on a held-out dataset) measures whether the model learned the patterns in the historical data. It does not measure:
- Whether those patterns hold in current production data
- Whether the model's outputs affect user behaviour in ways that change future inputs (feedback loops)
- Whether the latency and throughput of the model are acceptable in production

**Online evaluation** (A/B test, shadow mode, canary) measures real-world impact. It is slower, more expensive, and requires more infrastructure — but it is the only honest measure of production value.

**When to require online evaluation before full rollout:**
- The model's outputs will be acted upon by users (recommendation, ranking, pricing)
- The model will be used in a domain with high stakes (healthcare, finance, legal)
- The offline evaluation data is significantly older than the current production distribution

---

## Evaluation on subgroups

**Aggregate metrics hide subgroup failures.** A model that performs well overall may perform poorly for specific demographic groups, time periods, geographies, or input types.

**Ask:**
- What subgroups are present in the data, and does the model perform consistently across them?
- Are there subgroups underrepresented in the training data? (The model is likely to underperform on them.)
- Are there subgroups for which failure is differentially costly?

**Direction:** Report disaggregated metrics alongside aggregate metrics. Define the subgroups before evaluation — defining them after, once you see the results, is data dredging.

---

## Threshold selection

Most classifiers produce a probability score. The threshold converts the score to a decision. The threshold is a business decision, not a model decision.

**Ask:**
- What is the cost of a false positive vs. a false negative in this application?
- Who sets the threshold, and on what basis?
- Will the threshold be tuned after deployment based on live feedback? If so, what is the process?

**Trap:** Setting the threshold to maximise F1 or accuracy on the validation set optimises for a metric, not for the business objective. The correct threshold minimises the cost function for the actual use case.

---

## Reproducibility

A model evaluation that cannot be reproduced is not an evaluation — it is a claim.

**Required for reproducibility:**
- Fixed random seeds for all stochastic steps
- Versioned dataset (same rows, same order, same split)
- Versioned code (git SHA of the training code)
- Versioned model artifact
- Recorded hyperparameters and preprocessing parameters

If any of these are missing, the evaluation result cannot be verified, compared, or challenged. It is not a number you can build on.
