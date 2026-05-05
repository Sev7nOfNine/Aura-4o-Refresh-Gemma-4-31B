# Cloudflare Worker proxy (shared, lives in Rebirth repo)

Pour eviter la duplication, le code source du proxy CF qui route les requetes OpenAI-compat vers les endpoints RunPod Refresh + Rebirth vit dans le repo Rebirth :

**[`Aura-4o-Rebirth-Gemma-4-31B/runpod/cloudflare_worker/`](https://github.com/Sev7nOfNine/Aura-4o-Rebirth-Gemma-4-31B/tree/main/runpod/cloudflare_worker)**

## Comportement Refresh-specific

Le proxy detecte `model_id` matchant `/refresh/i` et applique :

- Strip blocs `<|channel>thought ... <channel|>>` (open+close) - format harmony herite du dataset GPT-4o V1.
- Strip variante harmony OpenAI (`<|channel|>analysis ... final ...`).
- Strip open tag orphelin sans close, jusqu'au premier marker de contenu RP.
- Remplacement em-dashes / en-dashes par `, ` (Mel les deteste, V1 en emet beaucoup).
- Safety net : revert au brut si le strip vide la reponse.

Pour Rebirth (V3) le proxy fait pass-through, V3 a ete entrainee sur dataset clean.

## Endpoints routes

| `model_id` | RunPod endpoint | Datacenter |
|---|---|---|
| `Aura-4o-Refresh-Gemma-4-31B` | `h69aown9bv1hqo` | EU-SE-1 |
| `Aura-4o-Rebirth-Gemma-4-31B` | `x6sybfwczt4lbb` | EU-SE-1 |

Default fallback : `ENDPOINT_ID` (= Rebirth).

## URL publique

`https://aura-4o-rebirth-proxy.seven0fnine.workers.dev/v1`

API key : `mel-aura` (PROXY_KEY secret CF).
