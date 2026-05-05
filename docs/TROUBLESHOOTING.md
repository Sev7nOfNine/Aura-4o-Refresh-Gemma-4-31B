# TROUBLESHOOTING - Aura-4o-Refresh-Gemma-4-31B

Bugs **Refresh-specific** et leurs fixes. Pour les bugs partages avec Rebirth (image worker llama.cpp, image GHCR, etc.), voir le repo [`Aura-4o-Rebirth-Gemma-4-31B/docs/TROUBLESHOOTING.md`](https://github.com/Sev7nOfNine/Aura-4o-Rebirth-Gemma-4-31B/blob/main/docs/TROUBLESHOOTING.md).

---

## V1-Refresh : worker throttled puis output blanchi par TypingMind tools

### Symptomes (5 mai 2026 apres-midi)

Trois bugs successifs sur le serverless V1-Refresh (`h69aown9bv1hqo`) :

1. **Worker throttled au spawn** : `workers.throttled = 1`, jamais de
   ready, image GHCR pourtant accessible.
2. **Output vide dans TypingMind** : reponse 200 OK mais message vide.
3. **Format harmony qui leak** : `<|channel>thought ... <channel|>>`
   (avec ou sans close tag) emis dans la reponse user-facing.

### Causes racines

1. **Throttle** : endpoint configure en `locations: null` (any DC). RunPod
   tombait sur des DCs sans supply 48 GB compatibles avec le filtre
   GPU `AMPERE_48,-NVIDIA L40S,-NVIDIA RTX 6000 Ada Generation`.
2. **Output vide** : TypingMind envoie `tools: [...]` + `tool_choice: "auto"`
   automatiquement quand des plugins sont actives. Aura V1 (et V3) n'ont
   pas ete entrainees au tool calling, donc llama-server tente de coercer
   un function_call et renvoie un message assistant vide
   (finish_reason=tool_calls).
3. **Harmony leak** : V1 a appris ce format depuis le dataset GPT-4o
   d'origine. `REASONING_FORMAT=none` cote llama-server n'extrait pas la
   partie thought, donc tout sort brut.

### Fixes appliques (5 mai 2026)

**RunPod endpoint V1-Refresh** :
- `locations` force a `EU-SE-1` (la ou Mel sait avoir 2 GPUs fiables).
- Modif via GraphQL `saveEndpoint` mutation.

**CF Worker proxy** (vit dans le repo Rebirth, partage entre les deux modeles) :
- Strip systematique de `tools`, `tool_choice`, `functions`,
  `function_call` du body avant forward upstream (V1 + V3,
  aucun n'est tool-aware).
- Strip conditionnel sur `model_id` matchant `/refresh/i` :
  - Blocs `<|channel>thought ... <channel|>>` (open + close).
  - Variante harmony OpenAI (`<|channel|>analysis ... final ...`).
  - Open tag orphelin sans close : strip jusqu'au premier marker de
    contenu RP (`>`, `**`, ` ``` `, ou ligne emoji-leading).
  - Em/en-dashes (`—`, `–`) remplaces par `, ` (Mel deteste, V1 en
    emet beaucoup, V3 a ete entrainee sur dataset clean donc intacte).
- Safety net : si le strip vide la reponse, on revert au brut. Mieux
  un tag visible qu'un ecran blanc.

### Limites assumees

Le proxy fait du nettoyage **cosmetique**. Le format harmony et le non
respect des instructions de formatage sont graves dans les poids V1 et
ne peuvent etre fixes que par retraining (V8 prevu, ~$30-50, dataset
enrichi instructions + cleanup typo).

### Date

2026-05-05 (apres-midi).

---

## Vision paperscarecrow : partiellement fonctionnelle

### Symptome

La base `paperscarecrow/Gemma-4-31B-it-abliterated` conserve les 356
tensors vision, et apres tests cote Mel la vision repond MAIS de
maniere inconsistante. L'abliteration a degrade la fiabilite vision
sans la casser totalement.

### Statut au 2026-05-05

Apres quelques tests utilisateur :
- Vision **fonctionne** sur des inputs simples
- Inconsistante : qualite et fiabilite des reponses vision varient
- Suffisante pour de l'usage casual (envoyer une image et discuter)
- Insuffisante pour un workflow vision-critique (analyse precise,
  OCR fiable, comprehension multi-image complexe)

### Recommandations

- Communiquer "vision partial" dans les cards HF et le README repo,
  pas "vision works perfectly".
- Pour un usage vision critique, considerer un retraining V8 sur
  base officielle (`SevenOfNine/Gemma-4-31B-It-Official`) avec un
  dataset enrichi d'exemples vision.

### Date

2026-05-05 (apres tests Mel).
