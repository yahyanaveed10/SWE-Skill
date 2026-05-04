---
name: ai-engineering
description: ML system design, MLOps pipeline signals, model evaluation discipline, and AI-specific failure modes. Use when designing data pipelines for ML, choosing between model serving strategies, diagnosing why a model works in notebooks but fails in production, evaluating model quality rigorously, or identifying hidden technical debt in ML systems. Does not cover general software architecture (see architecture-selection) or using AI models as a coding assistant (see ai-collaboration).
---

# AI Engineering

ML systems fail differently from conventional software systems. The model accuracy is often not the bottleneck — the data pipeline, serving infrastructure, and feedback loop from production to retraining are where production failures originate.

For ML pipeline design signals see [ml-pipeline-signals.md](ml-pipeline-signals.md).
For evaluation discipline see [evaluation-discipline.md](evaluation-discipline.md).
For MLOps failure modes see [mlops-failure-modes.md](mlops-failure-modes.md).

## The production gap

87% of ML models built do not reach production. Of those that do reach production, many degrade over time and are not caught until the degradation is visible to users. The gap is almost never model quality — it is engineering.

**Ask before building a model:** What data pipeline produces the training data? Who maintains it? Is the inference-time feature computation identical to the training-time feature computation? What monitoring exists for model output quality after deployment?

If those questions do not have clear answers, the model is not production-ready regardless of its validation metrics.

## The hidden technical debt frame

ML systems carry all the technical debt of conventional software plus:
- Data dependency debt (upstream schemas change, distributions shift)
- Feedback loop debt (the model's outputs affect the future inputs it will be evaluated on)
- Experimental code debt (exploratory notebooks promoted to production without engineering)
- Configuration debt (hyperparameters, thresholds, and feature sets as implicit state)

These debt types are invisible in standard code review and require ML-specific review practices.

## When ML is not the answer

Before designing an ML system, ask whether a simpler rule-based or heuristic system achieves the same goal with less operational overhead. ML adds:
- Training data collection and curation burden
- Model retraining cadence
- Feature engineering and pipeline maintenance
- Monitoring for data drift and model degradation
- Evaluation infrastructure

If a decision tree with five rules solves 90% of the cases, that is not a failure — it is the right engineering choice. Reserve ML for problems where the pattern space is too complex for explicit rules and enough labeled data exists to learn from.
