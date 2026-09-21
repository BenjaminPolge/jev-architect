# Example 2 — High-volume moderation

## Prompt

> We need to moderate user comments. Roughly 2 million a day. Block the abusive ones, and
> when we block something, tell the user why.

## What the skill does

| Step | Output | Closed? | Verdict |
|---|---|---|---|
| Strip spam already known by signature | match / no match | — | plain code |
| Is this comment abusive | probability | yes | Jev · `noul` |
| Severity | level on a scale | yes | Jev · `score` |
| Violation category | one of the policy categories | yes | Jev · `choice` |
| Explain the decision to the user | free text | no | LLM, only on appeal |

Two million decisions a day is exactly the regime where per-call LLM cost dominates the
budget, and the check sits in the publish path, so latency is user-visible.

## Expected output

```
**Proposed architecture**

- Known-signature spam filter → plain code
- Abusive yes/no → Jev (`noul`)
- Severity → Jev (`score`)
- Violation category → Jev (`choice`)
- Explanation to the user → LLM, on appeal only

| Criterion | Jev | LLM |
|---|---|---|
| Functional fit | ✔ binary decision + severity + category | ~ |
| Cost | ✔ input tokens only | ✘ 2M calls/day, prohibitive |
| Latency | ✔ fits inside the publish path | ✘ 1–3 s, visible on publish |
| Volume | ✔ target regime | ✘ |
| Determinism / typed output | ✔ threshold directly usable | ✘ |
| Free generation | ✘ | ✔ required for the explanation |
| Multi-step reasoning | ✘ | ✔ |
| Integration cost | ~ one more provider | ✔ already wired in |

**Why**: 2M decisions a day, a binary call with a threshold, a typed output usable as-is, and
latency the user feels at the moment they hit publish.

**Limit**: the uncertainty band around the threshold has to go somewhere — human review, or an
LLM if that slice stays small. Without that escape hatch, the cost saving is paid in false
positives.

This architecture, or keep everything on the LLM?
```

## The point

The **confidence gate** is what makes this architecture defensible: act on the confident
answers, escalate the uncertain band. Proposing the cost saving without the escalation path
would be a bad recommendation.
