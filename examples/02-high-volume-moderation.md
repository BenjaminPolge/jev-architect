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
**Architecture proposée**

- Filtre signatures connues → code classique
- Abus oui/non → Jev (`noul`)
- Gravité → Jev (`score`)
- Catégorie de violation → Jev (`choice`)
- Explication à l'utilisateur → LLM, uniquement sur contestation

| Critère | Jev | LLM |
|---|---|---|
| Adéquation fonctionnelle | ✔ décision binaire + gravité + catégorie | ~ |
| Coût | ✔ facturation à l'entrée uniquement | ✘ 2M appels/jour, prohibitif |
| Latence | ✔ compatible avec le chemin de publication | ✘ 1–3 s visibles à la publication |
| Volume | ✔ régime cible | ✘ |
| Déterminisme / sortie typée | ✔ seuil directement exploitable | ✘ |
| Génération libre | ✘ | ✔ nécessaire pour l'explication |
| Raisonnement multi-étapes | ✘ | ✔ |
| Complexité d'intégration | ~ un provider de plus | ✔ déjà en place |

**Pourquoi** : volume de 2M/jour, décision binaire à seuil, sortie typée exploitable
directement, et la latence est visible par l'utilisateur au moment de publier.

**Limite** : la bande d'incertitude autour du seuil doit partir en revue — humaine, ou LLM si
le volume concerné reste faible. Sans ce garde-fou, le gain de coût se paie en faux positifs.

Je pars là-dessus, ou on garde tout sur le LLM ?
```

## The point

The **confidence gate** is what makes this architecture defensible: act on the confident
answers, escalate the uncertain band. Proposing the cost saving without the escalation path
would be a bad recommendation.
