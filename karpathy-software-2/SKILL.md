---
name: karpathy-software-2
description: Use when deciding whether a problem should be solved with traditional code vs a neural network, when framing ML as an engineering discipline (not research), understanding dataset curation as software engineering, or evaluating the "is this worth MLing?" question. Applies Karpathy's Software 2.0 thesis.
---

# Karpathy — Software 2.0

## The Central Thesis (2017, still prescient)

> "The programmer of Software 2.0 does not write programs; they write datasets and loss functions. The weights are the program." — Karpathy, "Software 2.0"

Software 1.0: a developer specifies the rules explicitly. `if x > 0: do this`. Deterministic, interpretable, debuggable by reading the code.

Software 2.0: a developer specifies the *goal* (a loss function) and provides *examples* (a dataset). The optimizer finds the rules. The "code" is the learned weights — billions of floating-point numbers you can't read.

This is not just an architectural change. It changes what "programming" means, what "debugging" means, and what "quality engineering" means.

## The Two Programming Paradigms, Side by Side

| | Software 1.0 | Software 2.0 |
|---|---|---|
| **Specify by** | Writing rules (code) | Providing examples (data) |
| **Optimize with** | Human reasoning | Gradient descent |
| **Debug by** | Reading the code | Analyzing failure cases |
| **"Source code"** | `.py`, `.js`, `.rs` | Training dataset |
| **A bug is** | Wrong logic in code | Wrong/missing label in data |
| **Version control** | Git (code) | DVC / MLflow (data + models) |
| **Deployment artifact** | Binary / package | Model weights |
| **The compiler** | gcc / javac | Optimizer (Adam, SGD) |

## Data is Source Code — the Engineering Implication

If data is source code, then:

**Mislabeled data = a bug.** Not an unfortunate artifact — an actual defect that causes wrong behavior in production. The fix is what you'd do for a code bug: find it, fix it, prevent recurrence.

**Dataset curation = software engineering.** The practices that apply:
- **Version control the dataset.** Use DVC or similar. "What data was used to train v3.2.1?" must have an answer.
- **Write tests for the dataset.** Verify class balance, check for duplicates, validate label distributions, assert no test set contamination.
- **Code review the data pipeline.** The code that produces the dataset is as critical as the model code. Review it as carefully.
- **Treat label consistency as an API contract.** If "cat" and "kitten" are labeled differently by different annotators, your API is inconsistent.

```python
# A "data test" — should be in your CI
def test_no_test_contamination(train_ids, test_ids):
    assert len(set(train_ids) & set(test_ids)) == 0, "Data leakage: train/test overlap"

def test_class_balance(labels, max_imbalance_ratio=10):
    counts = Counter(labels)
    assert max(counts.values()) / min(counts.values()) < max_imbalance_ratio

def test_label_coverage(labels, expected_classes):
    assert set(labels) == set(expected_classes), f"Missing classes: {expected_classes - set(labels)}"
```

## Where Software 2.0 Wins Decisively

Not everything should be Software 2.0. The wins are specific:

| Problem | Why 2.0 wins |
|---|---|
| Image classification | The rules for "cat vs dog" cannot be written explicitly |
| Natural language understanding | Grammar is too complex to specify; examples are abundant |
| Recommendation | User preferences are latent, not articulable |
| Speech recognition | Acoustic→phoneme mapping is not hand-codeable |
| Protein folding | No human can write the rules; AlphaFold learned them |
| Game play (Atari, Go) | Strategy space too large for explicit rules |

Common thread: **the rules exist but cannot be written explicitly by a human.** Examples of correct behavior exist and are collectible. The objective (loss function) can be specified even if the rules can't.

## Where Software 1.0 Still Wins

```
Don't ML:
- When the rule is perfectly specifiable (sort, parse JSON, add two numbers)
- When you need perfect accuracy (financial transactions, safety-critical systems)
- When you have < 1000 labeled examples and can't get more
- When explainability is legally required and you can't satisfy it
- When a simple heuristic gets you 90% of the way with 10% of the effort
```

Karpathy's version: **try the stupidest possible thing first.** A linear classifier on hand-crafted features may solve your problem. If it does, ship it. Complexity has a cost that must be justified by results.

## The "Software 2.0 Stack"

What changes across the engineering stack when you adopt this paradigm:

```
1.0 stack                     2.0 stack
─────────────────────────     ─────────────────────────
IDE                           Jupyter + experiment tracker
Unit tests                    Data tests + eval harness
Code review                   Data audit + model card review
Profiler                      Loss curves + confusion matrix
Version control (git)         Data versioning (DVC) + model registry
Staging environment           Shadow deployment + canary release
Monitoring (error rate)       Monitoring (data drift + prediction dist)
Rollback (revert commit)      Rollback (reload previous model version)
```

The entire engineering toolchain has a 2.0 equivalent. Skipping any of them creates the same problems their 1.0 counterpart solved — you just encounter them in the model's behavior instead of in code.

## The Feedback Loop — Closing It

The most important operational practice in Software 2.0:

> **The model's predictions influence the world. The world generates the next dataset. The next dataset trains the next model.** This feedback loop is your most powerful asset and your most dangerous failure mode.

- **Asset version:** user interactions with your model generate labeled data (implicit feedback). A recommendation that was clicked is a positive label. Use it to retrain.
- **Failure mode:** the model promotes certain content → users see only that content → next dataset over-represents that content → model becomes more extreme. YouTube's recommendation history is the canonical case.

Engineering the feedback loop is a first-class product decision, not an afterthought.

**REQUIRED COMPANION:** karpathy-neural-nets covers the model-training side. karpathy-llm-intuitions covers applying this to LLMs specifically. For the ML engineering practices, see ml-engineer.
