# Example 5 — Audit of an existing project

## Prompt

> Our OpenAI bill has tripled this quarter. Have a look at the API layer and tell me what to do.

## What the skill does

Audit mode. It does not scan the whole repository on its own initiative — it looks at the call
sites in scope, and reports the ones whose output is a closed decision.

Here it finds this, called once per inbound document:

```python
resp = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": f"""
        Classify this document. Answer in JSON:
        {{"category": "invoice|contract|receipt|other",
          "priority": 1-5,
          "needs_review": true/false}}
        Document: {text}"""}],
)
data = json.loads(resp.choices[0].message.content)  # wrapped in try/except, fails ~1% of the time
```

Three signals, all pointing the same way: the output is entirely closed, the result is parsed
out of free text with a failure rate, and the call runs 40k times a day.

## Expected output

```
**Opportunity found** — `api/documents/classify.py:42`

This call produces three closed values (one category out of 4, a 1–5 priority, a boolean) but
goes through a generative model, with a `json.loads` that fails roughly 1% of the time.

- Category → Jev (`choice`)
- Priority → Jev (`score`)
- Needs review → Jev (`noul`)

All three questions are about the same state: one call.

| Criterion | Jev | LLM (current) |
|---|---|---|
| Functional fit | ✔ closed outputs | ~ works, format not guaranteed |
| Cost | ✔ input tokens only | ✘ the main line on the bill |
| Latency | ✔ ~100 ms | ✘ 1–3 s |
| Volume | ✔ 40k/day | ✘ |
| Determinism / typed output | ✔ no `json.loads`, no parse failures | ✘ ~1% failures |
| Free generation | ✘ not needed here | ✔ unused |
| Multi-step reasoning | ✘ not needed here | ✔ unused |
| Integration cost | ~ new provider + key + fallback | ✔ already wired in |

**Why**: none of the three outputs needs generation; the JSON parsing is a source of errors
that disappears with a typed response; 40k calls a day make the cost gap real.

**Limit**: this is a migration, not a drop-in swap — replay a sample of history and compare
decisions before cutting over, and keep the LLM path as a fallback.

Shall I prepare the migration for this call site, or do you want the sample comparison first?
```

## The point

Audit mode reports and proposes. It never rewrites the call site on its own, and it treats a
migration as something that needs to be validated against real history before shipping.
