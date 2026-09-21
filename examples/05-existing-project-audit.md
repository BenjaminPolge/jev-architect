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
**Opportunité détectée** — `api/documents/classify.py:42`

Cet appel produit trois valeurs fermées (catégorie parmi 4, priorité 1–5, booléen) mais passe
par un modèle génératif, avec un `json.loads` qui échoue environ 1 % du temps.

- Catégorie → Jev (`choice`)
- Priorité → Jev (`score`)
- Revue nécessaire → Jev (`noul`)

Les trois questions portent sur le même état : un seul appel.

| Critère | Jev | LLM (actuel) |
|---|---|---|
| Adéquation fonctionnelle | ✔ sorties fermées | ~ fonctionne, format non garanti |
| Coût | ✔ facturation entrée seule | ✘ poste principal de la facture |
| Latence | ✔ ~100 ms | ✘ 1–3 s |
| Volume | ✔ 40k/jour | ✘ |
| Déterminisme / sortie typée | ✔ plus de `json.loads`, plus d'échec de parsing | ✘ ~1 % d'échecs |
| Génération libre | ✘ non nécessaire ici | ✔ inutilisé |
| Raisonnement multi-étapes | ✘ non nécessaire ici | ✔ inutilisé |
| Complexité d'intégration | ~ nouveau provider + clé + fallback | ✔ déjà en place |

**Pourquoi** : aucune des trois sorties n'a besoin de génération ; le parsing JSON est une source
d'erreurs qui disparaît avec une réponse typée ; 40k appels/jour rendent l'écart de coût réel.

**Limite** : c'est une migration, pas un simple remplacement — il faut rejouer un échantillon
d'historique pour comparer les décisions avant de basculer, et garder le chemin LLM en fallback.

Je prépare la migration de ce call site, ou tu veux d'abord le comparatif sur un échantillon ?
```

## The point

Audit mode reports and proposes. It never rewrites the call site on its own, and it treats a
migration as something that needs to be validated against real history before shipping.
