# MLOps Failure Modes

Recurring failure patterns in ML systems that are not visible in notebooks or offline evaluations. These are the problems that cause 87% of models to never reach production, or to reach production and degrade silently.

---

## The notebook-to-production gap

A model that works in a Jupyter notebook is not a system. Notebooks:
- Execute cells in arbitrary order (hidden state)
- Have no versioning of intermediate results
- Mix data exploration, feature engineering, training, and evaluation in a single file
- Cannot be reliably reproduced without manual intervention

**The promotion problem:** Promoting a notebook to production means rewriting it as a pipeline — which introduces the risk that the reimplementation differs from the notebook in ways that matter.

**Signal:** The team says "we have a working model" and means "we have a working notebook." Ask: Is there a reproducible, version-controlled pipeline that produces the same artifact from the same inputs without manual steps? If not, the model is not production-ready.

---

## Model rot

The model was accurate at training time. Production conditions have changed (data distribution, user behaviour, upstream data schema). The model is now wrong but still serving.

**Signals:**
- No monitoring on model output quality (only infrastructure monitoring — uptime, latency)
- Retraining is manual and infrequent
- Business metrics are declining with no obvious cause

**Ask:**
- What monitoring exists for model output quality, not just model serving health?
- When was the model last retrained on fresh data?
- Is there a process to detect when performance has degraded below an acceptable threshold?

**Direction:** Monitor model outputs against ground truth labels with whatever lag is acceptable. Monitor input distribution drift as an early warning. Define a retraining trigger (time-based or drift-based).

---

## The feedback loop trap

The model's outputs affect the future data it is evaluated on. Common forms:

**Exposure bias:** A recommendation model recommends items → users click on recommendations → future training data is biased toward recommended items → the model learns to recommend what it already recommended, not what is genuinely useful.

**Performative prediction:** A fraud model scores transactions → flagged transactions are reviewed → only reviewed transactions get labels → the model is evaluated only on data it flagged (survivorship bias).

**Ask:**
- Does the model's decision affect what future training data looks like?
- Are there inputs the model never sees because they are filtered before reaching it?
- Is the evaluation data representative of the full input space, or only of inputs the model has already acted on?

---

## Undeclared dependencies

**Data dependency:** The model depends on a data source that changes without notice (schema changes, data quality changes, upstream pipeline failures). The model fails silently or degrades.

**Infrastructure dependency:** The model depends on a library version, GPU driver, or hardware configuration that is not declared. It works in development, fails in production.

**Glue code dependency:** 70-80% of ML system code is data pipeline, feature transformation, and infrastructure glue — not model code. This code has the same maintenance burden as any production system and is often less tested.

**Ask:**
- What upstream data sources does this model depend on? Are schema changes to those sources versioned and communicated?
- Is the full dependency stack (Python version, library versions, hardware requirements) pinned and reproducible?
- Is the glue code tested with the same rigor as the model evaluation?

---

## Hyperparameter and configuration debt

Hyperparameters and feature engineering choices are often implicit — decided once, never revisited, not tracked. Over time:
- The rationale is lost
- The values are tuned for a data distribution that no longer exists
- Changing them requires re-running experiments that were never documented

**Ask:**
- Are hyperparameters tracked alongside model versions?
- Is there a record of what experiments were run and what they showed?
- Are feature engineering choices documented with the rationale?

**Direction:** Track experiments (MLflow, Weights & Biases, or a simple spreadsheet). Version model artifacts with their associated hyperparameters and training dataset version. Treat configuration as code.

---

## Serving infrastructure failures

**Cold start latency:** A model loaded into memory on the first request creates unacceptable latency. Pre-warm the serving instance.

**Memory and compute underestimation:** A model that fits in a notebook does not fit in the serving container with the same resource allocation. Profile serving memory and CPU separately from training.

**Concurrency:** Notebooks are sequential. Production serving is concurrent. A model that uses global state, modifies input in-place, or is not thread-safe will produce incorrect results or crash under concurrent load.

**Serialisation format mismatch:** The model was saved in one format (pickle, joblib, SavedModel) and loaded in an environment that does not support it, or a different version of the library.

**Ask:** Has the model been tested in the serving environment — not on a developer laptop, not in a notebook, but in the exact container and hardware configuration that will serve production traffic — under realistic concurrent load?
