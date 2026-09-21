# Example 4 — Log analysis pipeline

## Prompt

> Build a pipeline that analyses our application logs and tells us what's going wrong.
> About 10 million lines a day.

## What the skill does

The first question is not "Jev or LLM" but "does this need a model at all". Ten million lines
a day collapse into a few thousand distinct signatures; only those need judgement.

| Step | Output | Closed? | Verdict |
|---|---|---|---|
| Parse and normalize lines | structured records | — | plain code |
| Group into signatures | ~2k distinct clusters/day | — | plain code |
| Is this signature actionable | true / false | yes | Jev · `noul` |
| Severity | level on a scale | yes | Jev · `score` |
| Area of the system | one of the known components | yes | Jev · `choice` |
| Write the incident summary | free text | no | LLM, on the top clusters only |

## Expected output

```
**Proposed architecture**

- Parsing and normalization → plain code
- Grouping into signatures (10M lines → ~2k clusters) → plain code
- Actionable yes/no → Jev (`noul`)
- Severity → Jev (`score`)
- Component affected → Jev (`choice`)
- Incident write-up → LLM, on the top clusters only

| Criterion | Jev | LLM |
|---|---|---|
| Functional fit | ✔ three closed outputs per signature | ~ |
| Cost | ✔ ~2k evaluations/day, negligible | ✘ if applied to raw lines |
| Latency | ✔ fits a near-real-time pass | ~ acceptable in batch |
| Volume | ✔ after grouping | ✘ before grouping |
| Determinism / typed output | ✔ severity usable for alerting | ✘ |
| Free generation | ✘ | ✔ required for the write-up |
| Multi-step reasoning | ✘ | ✔ correlating incidents |
| Integration cost | ~ one more provider | ✔ already wired in |

**Why**: most of the volume should touch no model at all — grouping does the heavy lifting. On
the ~2k remaining signatures, the three questions are closed and fit in one call per signature.

**Limit**: correlating incidents ("these three signatures are the same problem") takes
reasoning and stays on the LLM.

Shall I build it this way?
```

## The point

The biggest win here is not Jev. It is the reduction from 10M lines to 2k signatures in plain
code. A skill that reached for a model on every line would be doing harm. Recommending plain
code is a valid outcome of the check.
