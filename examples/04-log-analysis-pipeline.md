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
**Architecture proposée**

- Parsing et normalisation → code classique
- Regroupement en signatures (10M lignes → ~2k clusters) → code classique
- Actionnable oui/non → Jev (`noul`)
- Gravité → Jev (`score`)
- Composant concerné → Jev (`choice`)
- Synthèse d'incident → LLM, sur les clusters prioritaires uniquement

| Critère | Jev | LLM |
|---|---|---|
| Adéquation fonctionnelle | ✔ trois sorties fermées par signature | ~ |
| Coût | ✔ ~2k évaluations/jour, négligeable | ✘ si appliqué aux lignes brutes |
| Latence | ✔ compatible avec un passage temps quasi réel | ~ acceptable en batch |
| Volume | ✔ après regroupement | ✘ avant regroupement |
| Déterminisme / sortie typée | ✔ gravité exploitable pour l'alerting | ✘ |
| Génération libre | ✘ | ✔ nécessaire pour la synthèse |
| Raisonnement multi-étapes | ✘ | ✔ corrélation entre incidents |
| Complexité d'intégration | ~ un provider de plus | ✔ déjà en place |

**Pourquoi** : l'essentiel du volume ne doit toucher aucun modèle — le regroupement fait le gros
du travail. Sur les ~2k signatures restantes, les trois questions sont fermées et tiennent dans
un seul appel par signature.

**Limite** : la corrélation entre incidents (« ces trois signatures sont le même problème »)
demande du raisonnement et reste sur le LLM.

Je pars là-dessus ?
```

## The point

The biggest win here is not Jev. It is the reduction from 10M lines to 2k signatures in plain
code. A skill that reached for a model on every line would be doing harm. Recommending plain
code is a valid outcome of the check.
