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
**Architecture proposée**

- Score de fit ICP → Jev (`score`, 5 niveaux)
- Segment → Jev (`choice`)
- Budget évoqué / décideur atteint → Jev (`noul` ×2)
- Extraction du montant → parsing ou LLM (valeur libre, hors périmètre Jev)
- Rédaction de l'email → LLM

| Critère | Jev | LLM |
|---|---|---|
| Adéquation fonctionnelle | ✔ sur les 4 décisions fermées | ✔ sur la rédaction |
| Coût | ✔ qualification à bas coût | ~ acceptable, volume d'emails plus faible |
| Latence | ✔ qualification quasi instantanée | ~ rédaction hors chemin critique |
| Volume | ✔ toute la base qualifiée en lot | ✘ à ce volume |
| Déterminisme / sortie typée | ✔ score et segment directement stockables en CRM | ✘ |
| Génération libre | ✘ | ✔ indispensable |
| Raisonnement multi-étapes | ✘ | ✔ |
| Complexité d'intégration | ~ un provider de plus | ✔ déjà en place |

**Pourquoi** : les quatre décisions sont fermées et partagent le même état (une seule requête) ;
la base entière peut être requalifiée en lot sans exploser le budget ; le CRM reçoit des valeurs
typées plutôt que du JSON à valider.

**Limite** : le montant du budget est une valeur libre — Jev ne peut pas la produire. Elle
reste au parsing ou au LLM.

Je pars là-dessus ?
```

## The point

A feature that needs generation is not a feature that needs generation *everywhere*. Qualify
on Jev, write on the LLM.
