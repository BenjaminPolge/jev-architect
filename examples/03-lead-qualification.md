# Example 3 — Lead qualification (classification *and* generation)

## Prompt

> Add a lead qualification engine: read the CRM notes, tell us how good the lead is, which
> segment it belongs to, and write a personalized first outreach email.

## What the skill does

This feature genuinely needs both engines. The value of the skill here is drawing the line in
the right place, not picking a side.

| Step | Output | Closed? | Verdict |
|---|---|---|---|
| Fit with the ideal customer profile | level on a 1–5 scale | yes | Jev · `score` |
| Segment | one of `smb`, `mid-market`, `enterprise` | yes | Jev · `choice` |
| Budget mentioned in the notes | true / false | yes | Jev · `noul` |
| Decision-maker reached | true / false | yes | Jev · `noul` |
| Extract the stated budget amount | a number in free text | **no** | parser or LLM |
| Write the outreach email | free text | no | LLM |

Note the fifth row. "Is a budget mentioned" is a closed question and fits Jev. "What is the
amount" is not — Jev only returns values you enumerated, so an arbitrary figure is out of
scope. This distinction is the one most often got wrong.

## Expected output

```
**Proposed architecture**

- ICP fit score → Jev (`score`, 5 levels)
- Segment → Jev (`choice`)
- Budget mentioned / decision-maker reached → Jev (`noul` ×2)
- Extracting the amount → parser or LLM (free-form value, outside Jev's scope)
- Email drafting → LLM

| Criterion | Jev | LLM |
|---|---|---|
| Functional fit | ✔ on the 4 closed decisions | ✔ on the drafting |
| Cost | ✔ cheap qualification | ~ acceptable, email volume is lower |
| Latency | ✔ near-instant qualification | ~ drafting is off the critical path |
| Volume | ✔ whole base re-qualified in batch | ✘ at that volume |
| Determinism / typed output | ✔ score and segment stored straight into the CRM | ✘ |
| Free generation | ✘ | ✔ indispensable |
| Multi-step reasoning | ✘ | ✔ |
| Integration cost | ~ one more provider | ✔ already wired in |

**Why**: the four decisions are closed and share the same state (one request); the whole base
can be re-qualified in batch without blowing the budget; the CRM receives typed values instead
of JSON to validate.

**Limit**: the budget amount is a free-form value — Jev cannot produce it. It stays with a
parser or the LLM.

Shall I build it this way?
```

## The point

A feature that needs generation is not a feature that needs generation *everywhere*. Qualify
on Jev, write on the LLM.
