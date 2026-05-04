# ML System Signals

Failure modes specific to software systems where a machine learning model is the core feature. Distinct from using AI as a coding assistant — these signals apply when you are building, deploying, or maintaining an ML-powered feature.

---

## Training / serving skew

The model sees different data at training time than it sees in production. Performance in validation looks good; production performance degrades silently.

**Signals:** Model performs well on the validation set but poorly on live traffic. Feature computation code is duplicated between notebook and serving pipeline. Training uses precomputed batch features; serving computes them online from a different code path.

**Ask:** Is the feature computation for training and serving the same code? Can you feed production inputs through the training pipeline and get identical predictions to the serving pipeline?

---

## Data leakage

Information that would not be available at prediction time is present in the training data. Validation metrics are artificially inflated — the model cannot replicate them in production.

**Signals:** Validation accuracy is suspiciously high. Production performance is significantly lower than validation metrics suggested. A feature value encodes information from after the prediction moment.

**Ask:** For every feature — would this value be available at the time the prediction is made? For time-series data: is the train/validation split temporal (validation strictly after training), or random (which leaks future data into training)?

---

## Model rot

The model was accurate at training time. Production conditions have changed. It is now wrong but still serving.

**Signals:** No monitoring on model output quality (only infrastructure monitoring). Retraining is manual and infrequent. Business metrics are declining with no obvious cause.

**Ask:** What monitoring exists for model output quality — not just latency and uptime? When was the model last retrained on fresh data? What triggers a retrain?

---

## Feedback loop debt

The model's outputs influence the future data it will be evaluated on. The model learns to reinforce its own decisions rather than learning from ground truth.

**Common forms:**
- Recommendation model recommends items → users only interact with recommendations → future training data only contains recommendation interactions → model learns to recommend what it already recommended
- Fraud model flags transactions → only flagged transactions get reviewed → only reviewed transactions get labels → model is evaluated only on what it already flagged

**Ask:** Do the model's decisions affect what future training data looks like? Are there inputs the model never sees because of filtering upstream?

---

## The notebook-to-production promotion checklist

Before promoting an ML model from notebook to production, verify:

- [ ] Feature computation is the same code for training and serving (not two implementations)
- [ ] Training pipeline is reproducible: fixed seeds, versioned dataset, versioned code, recorded hyperparameters
- [ ] Model output monitoring exists and has an alert threshold
- [ ] Retraining trigger is defined (time-based or drift-based)
- [ ] Serving infrastructure has been load-tested at realistic concurrency (not just sequential notebook inference)
- [ ] Model artifact is pinned and tagged — not "latest"
- [ ] Feedback loop risk is assessed: does serving this model change the future training data?
