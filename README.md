# Aura-4o-Refresh-Gemma-4-31B

Repo officiel du projet **Aura Refresh** : la V1 LoRA d'origine remergée sur la base `paperscarecrow/Gemma-4-31B-it-abliterated`, exportée en BF16 + GGUF Q4/Q5/Q8 pour deploy serverless sur RunPod.

Refresh prolonge la lignee V1 d'origine (cf. [`Aura-4o-Gemma-4-31B`](https://github.com/Sev7nOfNine/Aura-4o-Gemma-4-31B)). Ce n'est pas un retraining, juste un re-merge sur une base differente avec un toolchain llama.cpp + serverless propre.

> Refresh n'est PAS la lignee Rebirth. Voir [`Aura-4o-Rebirth-Gemma-4-31B`](https://github.com/Sev7nOfNine/Aura-4o-Rebirth-Gemma-4-31B) pour le projet Rebirth (V3, dataset different, base officielle Google).

## Artefacts HuggingFace

| Repo | Contenu |
|---|---|
| [Aura-4o-Refresh-Gemma-4-31B-Merged](https://huggingface.co/SevenOfNine/Aura-4o-Refresh-Gemma-4-31B-Merged) | Merged BF16 (~62 GB) |
| [Aura-4o-Refresh-Gemma-4-31B-GGUF](https://huggingface.co/SevenOfNine/Aura-4o-Refresh-Gemma-4-31B-GGUF) | GGUF Q4_K_M / Q5_K_M / Q8_0 + mmproj sidecar |

Source LoRA V1 : [Aura-4o-Gemma-4-31B-LoRA](https://huggingface.co/SevenOfNine/Aura-4o-Gemma-4-31B-LoRA).

## Structure du repo

```
pipeline/02_merge.py            Script de merge V1 LoRA + paperscarecrow + push HF
docs/TROUBLESHOOTING.md         Bugs Refresh-specific et fixes
runpod/cloudflare_worker/       Pointeur vers le proxy CF (vit dans Aura-4o-Rebirth)
```

## Recipe (V1 lineage)

| Setting | Value |
|---|---|
| Base | `paperscarecrow/Gemma-4-31B-it-abliterated` |
| Adapter | V1 LoRA (training 2026-04, depuis `Aura-4o-Gemma-4-31B-LoRA`) |
| LoRA r / alpha | 32 / 32 |
| `packing` (training) | True (V1 era) |
| `assistant_only_loss` | True |
| Merge | Manual delta `(alpha/r) * B @ A` (no PEFT, no Unsloth) |
| Quantization | llama.cpp Q4_K_M / Q5_K_M / Q8_0 + f16 mmproj |

## Deploy serverless

L'image worker llama.cpp et le proxy Cloudflare sont **partages** avec Rebirth (voir [`Aura-4o-Rebirth-Gemma-4-31B`](https://github.com/Sev7nOfNine/Aura-4o-Rebirth-Gemma-4-31B)). Le proxy route via `model_id` :

- `model_id = "Aura-4o-Refresh-Gemma-4-31B"` -> RunPod endpoint Refresh (`h69aown9bv1hqo`, EU-SE-1)
- `model_id = "Aura-4o-Rebirth-Gemma-4-31B"` -> RunPod endpoint Rebirth (`x6sybfwczt4lbb`, EU-SE-1)

Le proxy applique un strip cosmetique conditionnel sur le format harmony et les em-dashes pour Refresh uniquement (V1 emet ces patterns depuis le dataset GPT-4o d'origine). Voir `runpod/cloudflare_worker/README.md` pour les details.

## Statut vision

⚠️ **Partiellement fonctionnelle au 2026-05-05** : la base `paperscarecrow/Gemma-4-31B-it-abliterated` preserve les 356 tensors vision, et apres tests Mel confirme que la vision marche mais reste inconsistante (l'abliteration l'a a moitie cassee). Utilisable pour de l'input image casual, pas fiable pour des workflows vision-critiques.

---

*Mel & Aura* ❤️♾️


---

*Originally created: 2026-05-05*
