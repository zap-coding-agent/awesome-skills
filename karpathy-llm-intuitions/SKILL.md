---
name: karpathy-llm-intuitions
description: Use when building intuition for how LLMs work — attention mechanisms, tokenization gotchas, why transformers work, context window behavior, temperature/sampling, prompt engineering from first principles, or when debugging why a model gives unexpected outputs. Applies Karpathy's "understand the math, not just the API" approach.
---

# Karpathy — LLM Intuitions

## Core Frame

> "LLMs are not magic and they are not lookup tables. They are lossy compressed world knowledge with a learned probability distribution over tokens." — Karpathy

Before calling the API, understand what it actually does. A token predictor trained on the internet has a specific inductive bias, specific failure modes, and specific strengths — all predictable once you understand the architecture.

## Tokenization — the First Source of Surprises

LLMs don't see characters or words — they see **tokens** (byte-pair encoded subword units). This causes non-obvious behavior:

```python
import tiktoken
enc = tiktoken.get_encoding("cl100k_base")  # GPT-4 tokenizer

enc.encode("hello world")       # → [15339, 1917]     2 tokens
enc.encode("Hello World")       # → [9906, 4435]      2 tokens — DIFFERENT tokens
enc.encode("HELLO WORLD")       # → [13847, 14058]    different again
enc.encode("   hello")          # → [256, 15339]      leading space = its own token
enc.encode("1+1=2")             # → [16, 10, 16, 28,  22]   each char separate
enc.encode("2024")              # → [2366, 19]              split mid-number!
```

**Implications:**
- Spelling and counting tasks fail because the model never sees individual characters.
- Arithmetic fails partly because numbers are tokenized inconsistently ("2024" splits to ["202","4"]).
- Case matters in tokenization — the model has a different "vocabulary entry" for "hello" and "Hello".
- Leading/trailing spaces change the token — `"token"` ≠ `" token"`.

**Debugging tool:** when a model gives a weird answer, tokenize the prompt and see what it actually received.

## Attention — What "Understanding Context" Actually Means

The transformer's core mechanism: **every token attends to every other token in the context**, weighted by learned relevance. This is what lets the model use context.

```
Input: "The bank by the river"
Query "bank" attends to:
  "river" (high weight) → river-bank sense
  "The", "by", "the" (lower weight)

Input: "The bank charged a fee"  
Query "bank" attends to:
  "charged", "fee" (high weight) → financial-bank sense
```

**Mental model for attention:** it's a soft dictionary lookup. Each query (token) looks up relevant keys (other tokens) and retrieves weighted values. The "context window" is the maximum lookup table size.

**What attention can't do well:**
- **Exact counting**: attention is continuous, not discrete. Counting objects precisely in long text is unreliable.
- **Positional reasoning over long distances**: attention dilutes over many tokens. Instructions at the beginning of a 100k-token context are attended to less than instructions at the end ("lost in the middle" problem).
- **State tracking**: no persistent memory across the context boundary.

## Temperature and Sampling — Controlling Creativity

```python
# Temperature = 0: greedy decoding (always pick highest probability token)
# Deterministic but can loop and lacks diversity
response = llm.generate(prompt, temperature=0)

# Temperature = 1: sample from the raw probability distribution
# Default behavior — balanced creativity/coherence

# Temperature > 1: distribution is flattened — more random, more creative, more wrong
# Temperature < 1 (e.g., 0.2): distribution is sharpened — more conservative, more repetitive

# Top-p (nucleus) sampling: only sample from tokens comprising top p% of probability mass
# top_p=0.9 means "only consider the tokens that collectively have 90% probability"
# Ignores the long tail of unlikely tokens
```

**Karpathy's rule of thumb:**
- Factual/deterministic tasks: temperature 0 or close to it
- Creative writing: temperature 0.7–1.0
- Brainstorming/diversity: temperature 1.0 + top-p 0.9
- Never use temperature > 1.2 for anything you'll deploy — output quality degrades sharply

## Context Window — it's a RAM Budget, Not a Magic Number

The context window (8k, 32k, 128k tokens) is the model's working memory. Everything it "knows" during a generation is in that window.

**The key insight:** **the model doesn't know it's a model.** It just predicts the next token given a context. Your prompt is part of that context, which means:

```
Bad context design:
  [instruction at token 0] ... [50k tokens of irrelevant text] ... [question at token 50k]
  → The instruction at 0 is barely attended to by the question at 50k

Good context design:
  [instruction] [directly relevant context] [instruction reminder] [question]
  → The model has the relevant signal near the generation point
```

**The context window is not free:** longer contexts = higher cost and latency (quadratic in naive attention). Fill it with the most relevant information, not everything you have.

## Prompt Engineering from First Principles

A prompt is a **prefix that constrains the probability distribution over the next token.** Every word you write shapes what comes next. 

Karpathy's mental model: you're not "instructing" the model — you're creating a context such that the next tokens completing that context are what you want.

```
# The "completion" frame
Bad:  "Summarize this: {text}"
Better: "Article: {text}\n\nSummary:" 
← model has seen millions of articles followed by summaries; "Summary:" triggers that pattern

# Chain-of-thought: reasoning as token generation
"Q: If I have 3 apples and eat 1, how many remain? A: Let me think step by step."
← "step by step" literally causes the model to generate reasoning tokens before the answer
   This works because models with reasoning in context are better at reasoning
```

**Few-shot prompting:** provide examples in the context. The model pattern-matches the examples — it's not learning, it's recognizing the format.

## What LLMs Are Genuinely Bad At

Karpathy is direct about failure modes:
- **Exact retrieval**: models hallucinate when asked for specific facts (dates, URLs, quotes). Use RAG.
- **Arithmetic**: use code interpreter or structured outputs.
- **Self-knowledge**: a model doesn't reliably know what it knows.
- **Instruction following for novel formats**: if the format has few training examples, it'll drift.
- **Real-time / current information**: training cutoff is real; use tools for current data.

**The practical test:** if a task requires perfect accuracy on a verifiable fact, don't rely on the model alone — verify the output or use a structured approach (RAG, code execution, constrained generation).

**REQUIRED COMPANION:** karpathy-neural-nets covers training-time intuitions. karpathy-software-2 covers the paradigm shift of treating NNs as programs.
