---
name: ml-engineer
description: Use when approaching problems as an ML engineer — framing ML problems correctly, choosing between ML vs non-ML solutions, designing training/evaluation pipelines, preventing data leakage, setting up experiment tracking, deploying models, and monitoring model quality in production. Covers the ML engineering mindset alongside the software engineering practices that make ML reliable.
---

# ML Engineer Mindset

## Core Philosophy

Machine learning is a **software engineering problem with probabilistic outputs.** The failures that kill ML projects aren't usually wrong model choices — they're wrong problem framing, untrustworthy evaluation, silent data pipeline bugs, and production drift that nobody monitors. 

> "The most common failure mode: we build the model, but the model isn't the product." — Josh Wills

Engineering rigor matters *more* in ML than in conventional software — because the feedback loops are slower, bugs are silent (wrong outputs, not crashes), and the number of ways to fool yourself during evaluation is enormous.

## When NOT to Use ML

Before building a model, ask: **can a simple rule solve this?**

```
❌ ML first, question second:
   "We should build a recommendation model for our 10 users"
   
✅ Question first, tool second:
   "Users can't find relevant content" → 
     Can we fix search? → Yes → fix search first.
     Can we fix taxonomy/tags? → Yes → try that.
     Does the problem require learning from behavior? → Then ML.
```

ML is the right tool when: the pattern is too complex to hand-code, examples of correct behavior are abundant, and the cost of being slightly wrong is tolerable. It's wrong when a rule or query does the job, data is scarce, or explainability is legally required and model isn't interpretable.

## Problem Framing: the Most Important Step

Bad framing → good model → wrong product. The questions:

1. **What exactly are we predicting?** (label definition)
2. **What features are available *at prediction time*?** (not at training time — data leakage)
3. **What does "wrong" cost?** (FP vs FN; calibration matters if you're ranking/thresholding)
4. **What is the business metric?** (does optimizing the ML metric actually improve the business metric?)
5. **Is there a feedback loop?** (model influences behavior influences future training data)

## Data Leakage — the Silent Killer

**Leakage**: information available during training that won't be available at prediction time, causing unrealistically good evaluation metrics and failure in production.

```python
# ❌ Leakage: scaling uses the full dataset including the test set
scaler.fit(X)                                    # sees test data
X_scaled = scaler.transform(X)
X_train, X_test = train_test_split(X_scaled)     # too late

# ✅ Fit on train only; transform both
X_train, X_test, y_train, y_test = train_test_split(X, y)
scaler.fit(X_train)                              # only train
X_train_s = scaler.transform(X_train)
X_test_s  = scaler.transform(X_test)             # apply same transform

# In a pipeline, this is automatic:
pipe = Pipeline([("scaler", StandardScaler()), ("clf", LogisticRegression())])
pipe.fit(X_train, y_train)   # fit and transform internally correct
```

Watch for temporal leakage in time-series: splitting randomly uses future data to predict the past. Always split by time for sequential data.

## Evaluation Design (before training)

1. **Define the metric before looking at results.** Choose F1 / AUC / NDCG / calibration based on the problem.
2. **Set a baseline** — random, majority class, most-recent, simple heuristic. A model that can't beat the baseline needs different framing, not more layers.
3. **Separate validation from test.** Validation tunes hyperparameters. Test is held out until the final evaluation — touch it once, at the end.
4. **Slice your evaluation.** Global accuracy hides demographic disparities, rare class failures, and distribution shift. Evaluate on subgroups, edge cases, and the hardest examples.

## Experiment Tracking

Every experiment must be reproducible. Log:

```python
# MLflow, W&B, or DVC — the tool doesn't matter; the habit does
with mlflow.start_run():
    mlflow.log_params({
        "model": "xgboost",
        "max_depth": 6,
        "n_estimators": 200,
        "train_date_range": "2023-01-01/2024-01-01",    # data provenance
        "feature_version": "v3",                         # which feature set
    })
    mlflow.log_metrics({"auc": 0.89, "f1": 0.74, "precision": 0.81})
    mlflow.log_artifact("model.pkl")
    mlflow.set_tag("hypothesis", "adding recency feature improves CTR prediction")
```

What to track: hyperparameters, data version + date range, feature set version, all metrics, model artifact, and the hypothesis being tested. Without this, you can't reproduce results or compare experiments reliably.

## Model Deployment Patterns

| Pattern | When |
|---|---|
| **Batch inference** | Predictions computed ahead of time; acceptable latency; large volume |
| **Online (REST) inference** | Real-time, per-request; latency-sensitive |
| **Streaming inference** | Predictions on events in a stream (Kafka); near-real-time |
| **Embedded / on-device** | No network round-trip; privacy requirements; edge |

**Shadow mode deployment**: run new model in parallel with old, log both outputs, compare offline before switching traffic. Catches distribution shift and edge case failures without user impact.

**Canary deployment**: send 5% of traffic to the new model; watch business metrics (not just ML metrics) for a defined window before promoting.

## Production Monitoring: the Work That Doesn't End

Models degrade silently. Monitor:

| Signal | What it catches |
|---|---|
| **Prediction distribution** | Model drift — outputs shifting without label feedback |
| **Feature distribution** | Data drift — inputs changing |
| **Business metric** | The reason the model exists |
| **Label feedback (if available)** | Actual performance vs evaluation |
| **Latency / throughput** | Operational health |

Set alerts on prediction distribution shift (PSI, KL divergence) and on the business metric degrading below baseline. Don't wait for user complaints.

**Retraining strategy:** scheduled (every N days), triggered (when drift exceeds threshold), or continuous (online learning). The right choice depends on how fast the world changes for your problem.
