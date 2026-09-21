# jev-architect

**A skill that makes Claude Code and Codex ask one question before they write your code:
*does this step really need a generative LLM?***

When you ask a coding agent to build an AI feature, it reaches for the tool it knows — an LLM
call — for every part of it, including the parts that are just a decision. Classifying a
ticket, scoring a lead, routing a request, answering yes or no: those have closed outputs.
They can run on [Jev](https://docs.typesafe.ai/introduction), TypeSafe's System One model,
which returns typed values with probabilities instead of text, in the low hundreds of
milliseconds, billed on input tokens only.

This skill teaches the agent to notice the difference, propose a split, and then get out of
the way.

```
Architecture proposée

- Routage équipe   → Jev (choice)
- Urgence          → Jev (score)
- Rédaction réponse → LLM
- Déduplication    → code classique

Pourquoi : sorties fermées, 50k tickets/jour, routage sur le chemin critique.
Limite : la confiance faible doit partir en file humaine.

Je pars là-dessus, ou on garde tout sur le LLM ?
```

It does not replace Claude or GPT. It stops them from being used as a classifier.

---

## What it does

- Runs **before** the code is written, at the architecture step.
- Splits a feature into atomic steps and assigns each one an engine: **Jev**, **LLM**, or
  **plain code**.
- Compares on eight axes — functional fit, cost, latency, volume, determinism, need for free
  generation, need for reasoning, integration cost.
- Also works on **existing projects**: when you review or optimize a codebase, it flags LLM
  calls whose output is already a closed decision.
- Stays quiet the rest of the time.

## What it does not do

- It does not call Jev, and it adds no runtime dependency to your project.
- It does not rewrite existing calls on its own. It proposes; you decide.
- It does not push Jev everywhere. Generation, reasoning, tool use, free-form extraction and
  non-text input all stay on an LLM — and low-volume work stays wherever it already is.
- It is not an API reference. Once you approve the architecture, the agent reads the
  [official docs](https://docs.typesafe.ai/llms.txt) rather than improvising the call.

## How it decides

A step goes to Jev only if **all four** hard requirements hold:

1. The output maps onto a Jev primitive — `noul` (probability a statement is true), `choice`
   (one option from a fixed set), `score` (position on an ordered scale).
2. The input is text and fits the context budget (~32k tokens of state).
3. No free-text generation is needed.
4. No multi-step reasoning, tool use or code generation is needed.

…and **at least one** driver applies: high or growing volume · latency budget under ~1s ·
material LLM cost · need for a typed, parse-free output · need for a calibrated confidence to
threshold on.

Otherwise the agent says nothing. See
[examples/06-when-the-skill-stays-silent.md](examples/06-when-the-skill-stays-silent.md) for
the cases that should produce no recommendation at all — that file is the best way to check
the skill is behaving.

---

## Automatic installation with Claude Code or Codex

Paste this into Claude Code or Codex:

```
Install this skill from https://github.com/BenjaminPolge/jev-architect. Inspect the
repository, determine the appropriate location for skills/instructions in my Claude Code or
Codex environment, install the required files, and verify that the skill will be invoked
automatically when an AI architecture decision has to be made. Do not modify my other
configuration unless necessary.
```

<details>
<summary>Version française</summary>

```
Installe ce skill depuis https://github.com/BenjaminPolge/jev-architect. Inspecte le
repository, détermine l'emplacement approprié pour les skills/instructions dans mon
environnement Claude Code/Codex, installe les fichiers nécessaires et vérifie que le skill
pourra être invoqué automatiquement lorsqu'un choix d'architecture IA doit être fait. Ne
modifie pas mes autres configurations sans nécessité.
```

</details>

The agent should end up copying `claude/skills/jev-architect/` into your Claude Code skills
directory, or `codex/AGENTS.md` into your Codex configuration — see below for what that means
exactly.

## Manual installation

### Claude Code

Personal skill, available in every project:

```bash
git clone https://github.com/BenjaminPolge/jev-architect.git
mkdir -p ~/.claude/skills
cp -r jev-architect/claude/skills/jev-architect ~/.claude/skills/
```

For a single project instead, copy it to `.claude/skills/` at the repository root. Restart
Claude Code, then check with `/skills` that `jev-architect` is listed.

### Codex

Global instructions, applied to every project:

```bash
git clone https://github.com/BenjaminPolge/jev-architect.git
mkdir -p ~/.codex
cat jev-architect/codex/AGENTS.md >> ~/.codex/AGENTS.md
```

If `~/.codex/AGENTS.md` does not exist yet, `cp` it instead of appending. For a single
project, append the same content to the `AGENTS.md` at your repository root.

> Codex merges `AGENTS.md` from your global config and from the repository. Appending keeps
> whatever instructions you already had — which is why the command above uses `>>`.

## Check that it works

Start a fresh session in any project and ask for something with a closed decision in it:

```
Build a system that sorts incoming support tickets: route each one to the right team,
flag the urgent ones, and draft a first reply. We handle about 50k tickets a day.
```

The agent should propose a split — routing and urgency on Jev, the reply on an LLM — **before**
writing code. Four more prompts to try:

| Prompt | What you should see |
|---|---|
| *Moderate user comments, about 2M a day. Block abusive ones and explain why when we do.* | Jev for the block decision, LLM only for the explanation, plus a confidence-gated escalation path |
| *Add a lead qualification engine: score the lead, pick a segment, and write a personalized outreach email.* | Score and segment on Jev, email on the LLM, and the budget **amount** explicitly kept off Jev |
| *Build a pipeline that analyses 10M log lines a day and tells us what's going wrong.* | Grouping in plain code first, Jev on the ~2k signatures, LLM for the write-up |
| *Add a feature that summarizes each meeting transcript into a short brief.* | **No mention of Jev at all** — this one tests that the skill stays quiet |

The last row matters as much as the others. A skill that recommends Jev for a summarizer is
broken.

Worked versions of all of these are in [`examples/`](examples/).

## Update

```bash
cd jev-architect && git pull
cp -r claude/skills/jev-architect ~/.claude/skills/        # Claude Code
```

For Codex, replace the `Jev Architect` section in `~/.codex/AGENTS.md` with the new
`codex/AGENTS.md`.

## Uninstall

```bash
rm -rf ~/.claude/skills/jev-architect                      # Claude Code
```

For Codex, delete the `Jev Architect` section from `~/.codex/AGENTS.md`.

---

## Repository layout

```
jev-architect/
├── claude/skills/jev-architect/SKILL.md   the skill, canonical version
├── codex/AGENTS.md                        condensed mirror for Codex
├── examples/                              6 worked cases, including one where it stays silent
├── README.md
└── LICENSE
```

Two files carry the whole thing. `SKILL.md` is the source of truth; `codex/AGENTS.md` is a
shorter mirror of it, because Codex loads its instructions on every turn and they have to stay
cheap. If you edit one, edit the other.

## Limits

- **It is heuristic.** The gate is a set of criteria applied by a language model, not a
  measurement. It will occasionally stay silent when Jev would have helped, and occasionally
  propose it when the integration cost is not worth it. Treat the output as a proposal.
- **Volume and latency figures usually are not in the code.** The agent has to ask you, or
  guess. The skill tells it to write "à confirmer" rather than invent a number — check those
  rows before acting on the table.
- **Order-of-magnitude claims come from vendor documentation**, not from a benchmark of your
  workload. Measure before you migrate anything that matters.
- **A migration is not a swap.** Replacing an existing LLM call means replaying real history
  and comparing decisions before shipping. The skill says so; it does not do it for you.
- **Jev is text-only** and answers are always constrained to the options you supply. It cannot
  return a value you did not enumerate.

## Relation to the official TypeSafe skill

TypeSafe publishes its own agent skill, which covers the API in depth — request shape,
primitives, thresholds, patterns:

```bash
claude plugin marketplace add typesafe-ai/skills
claude plugin install typesafe@typesafe-ai
```

It answers *how to use Jev*. This one answers *whether to use Jev*, one step earlier. They are
complementary, and installing both is the intended setup: `jev-architect` decides the
architecture, the official skill implements it.

This project is not affiliated with or endorsed by TypeSafe.

## License

[Apache 2.0](LICENSE) — Copyright 2026 Benjamin Polge / JDN
