# Example 1 — Support ticket triage

## Prompt

> Build a system that sorts incoming support tickets: send each one to the right team, flag
> the urgent ones, and draft a first reply. We get about 50k tickets a day.

## What the skill does

Decompose first, then apply the gate to each step.

| Step | Output | Closed? | Verdict |
|---|---|---|---|
| Detect duplicate of an open ticket | ticket id or none | — | plain code (hashing / vector lookup) |
| Route to a team | one of `billing`, `technical`, `sales`, `account` | yes | Jev · `choice` |
| Urgency | level on a 1–4 scale | yes | Jev · `score` |
| Is a refund being requested | true / false | yes | Jev · `noul` |
| Draft the first reply | free text | no | LLM |

Three closed steps, 50k tickets a day, and the routing sits in a latency-sensitive path. The
gate passes. The three questions target the same state, so they travel in one request.

## Expected output

```
**Proposed architecture**

- Deduplication → plain code (hash + lookup)
- Team routing → Jev (`choice`)
- Urgency → Jev (`score`, 4 levels)
- Refund requested → Jev (`noul`)
- First reply drafting → LLM

| Criterion | Jev | LLM |
|---|---|---|
| Functional fit | ✔ closed outputs (team, level, yes/no) | ~ capable, but oversized |
| Cost | ✔ input tokens only, an order of magnitude lower | ✘ 50k tickets/day × 3 decisions |
| Latency | ✔ low hundreds of ms, 3 questions in one call | ✘ 1–3 s per call |
| Volume | ✔ 50k/day, squarely in range | ✘ cost scales linearly |
| Determinism / typed output | ✔ constrained value, no parsing | ✘ JSON to parse and validate |
| Free generation | ✘ impossible | ✔ required for the reply |
| Multi-step reasoning | ✘ | ✔ |
| Integration cost | ~ one more provider and key | ✔ already wired in |

**Why**: the three decisions have closed outputs and share the same state, so they travel in a
single call; the volume makes triage cost material; routing sits on the critical path.

**Limit**: reply drafting stays on the LLM, and low-confidence answers need a fallback — route
those to a human triage queue.

This architecture, or keep everything on the LLM?
```

## The point

The reply generation stays on the LLM. Only the decisions move. This is the normal shape of a
Jev recommendation: a hybrid, never a replacement.
