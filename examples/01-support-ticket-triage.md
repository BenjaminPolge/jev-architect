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
**Architecture proposée**

- Déduplication → code classique (hash + lookup)
- Routage équipe → Jev (`choice`)
- Urgence → Jev (`score`, 4 niveaux)
- Demande de remboursement → Jev (`noul`)
- Rédaction de la première réponse → LLM

| Critère | Jev | LLM |
|---|---|---|
| Adéquation fonctionnelle | ✔ sorties fermées (équipe, niveau, oui/non) | ~ capable, mais surdimensionné |
| Coût | ✔ tokens d'entrée seuls, ordre de grandeur inférieur | ✘ 50k tickets/jour × 3 décisions |
| Latence | ✔ une centaine de ms, 3 questions en un appel | ✘ 1–3 s par appel |
| Volume | ✔ 50k/jour, dans la cible | ✘ coût linéaire |
| Déterminisme / sortie typée | ✔ valeur contrainte, aucun parsing | ✘ JSON à parser et à valider |
| Génération libre | ✘ impossible | ✔ nécessaire pour la réponse |
| Raisonnement multi-étapes | ✘ | ✔ |
| Complexité d'intégration | ~ un provider et une clé de plus | ✔ déjà en place |

**Pourquoi** : les trois décisions ont des sorties fermées et partagent le même état, donc un
seul appel ; le volume rend le coût du tri significatif ; le routage est sur le chemin critique.

**Limite** : la rédaction de la réponse reste sur le LLM, et il faut un fallback si Jev répond
avec une confiance faible — dans ce cas, router vers une file de triage humain.

Je pars sur cette architecture, ou on garde tout sur le LLM ?
```

## The point

The reply generation stays on the LLM. Only the decisions move. This is the normal shape of a
Jev recommendation: a hybrid, never a replacement.
