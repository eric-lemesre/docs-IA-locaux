# Classement — Modèles locaux pour le **code agentique**

> Cible matérielle : 16 GB VRAM + 128 GB RAM. Inférence via Ollama.
> Mise à jour : avril 2026.

Le **code agentique** = boucle outil → action → résultat → décision. Exige : `tool calling` fiable, gros contexte, vitesse, suivi d'instructions strict.

---

## 🏆 Top 5 — Code agentique sur 16 GB VRAM

| # | Modèle | Taille | Active | SWE-bench | Verdict |
|---|---|---|---|---|---|
| 1 | **Qwen3-Coder 30B-A3B** | 30B MoE | 3B | **70.6 %** | 🥇 Référence absolue |
| 2 | **Devstral Small 2** | 24B dense | 24B | 68 % | 🥈 Spécialisé agentique pur |
| 3 | **GPT-OSS 20B** | 20B MoE | 5.1B | 52 %* | 🥉 Meilleur compromis vitesse |
| 4 | **Codestral 25.12** | 22B dense | 22B | ~60 % | Solide, codage classique |
| 5 | **Qwen2.5-Coder 7B** | 7B dense | 7B | 47 % | Léger, ultra-rapide |

\* GPT-OSS = score Intelligence Index (52.1 %), pas SWE-bench direct.

---

## 1. Qwen3-Coder 30B-A3B — 🥇 le meilleur

**Architecture MoE** : 30B paramètres totaux mais seulement **3B activés par token** → tient en 16 GB VRAM avec un gros contexte, et l'inférence est rapide comme un modèle 3B dense.

### Forces
- **#1 sur SWE-rebench Pass@5** (64.6 %), devant Claude Opus 4.6 (58.3 %)
- Mode `instruct` direct (pas de `<think>` blocks → latence faible pour boucles agentiques)
- Tool calling natif testé sur Cursor / Cline / OpenCode
- Vient avec **Qwen Code**, son propre CLI agentique

### Installation
```bash
# Quants Unsloth recommandés (incluent les fixes tool calling)
ollama pull hf.co/unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF:UD-Q4_K_XL

# Variante contexte étendu (essentielle)
ollama run hf.co/unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF:UD-Q4_K_XL
>>> /set parameter num_ctx 32768
>>> /save qwen3-coder:30b-a3b-32k
```

### Limitations
- Format XML non standard pour les tool calls → exige `llama.cpp` ≥ mars 2026
- Bug `key_gdiff` dans les anciens builds → boucle infinie

---

## 2. Devstral Small 2 — 🥈 spécialiste agentique

24B dense par Mistral, conçu **exclusivement** pour les workflows agentiques (OpenHands, SWE-Agent, Aider).

### Forces
- 68 % SWE-bench Verified — bat DeepSeek-V3 671B et Qwen3 232B
- Optimisé pour résoudre des **issues GitHub de bout en bout**
- Comprend la structure d'un repo entier sans guidance

### Installation
```bash
ollama pull devstral-small-2

# Contexte étendu (24k tient en 16 GB Q4)
ollama run devstral-small-2
>>> /set parameter num_ctx 24576
>>> /save devstral-small-2:24k
```

### Quand l'utiliser plutôt que Qwen3-Coder
- Si ton workflow ressemble à du SWE-bench (issue → fix → PR)
- Si tu utilises OpenHands ou Aider
- Si tu veux du dense (latence par token plus régulière qu'un MoE)

---

## 3. GPT-OSS 20B — 🥉 meilleur compromis vitesse

MoE OpenAI (20B / 5.1B actifs) avec quantization MXFP4. **Le plus rapide** des modèles compétents sur 16 GB.

### Forces
- **42+ tokens/s** quel que soit le contexte (jusqu'à 60k)
- Score logique parfait sur les tests cognitifs
- Apache 2.0 → utilisation commerciale libre
- Excellent pour les boucles **interactives rapides**

### Installation
```bash
ollama pull gpt-oss

ollama run gpt-oss
>>> /set parameter num_ctx 65536
>>> /save gpt-oss:60k
```

### Quand le préférer
- Boucle agentique itérative avec beaucoup d'aller-retours
- Refactor sur très large contexte (50k+ tokens)
- Quand la vitesse > la performance brute SWE-bench

---

## 4. Codestral 25.12 — codage dense fiable

22B dense par Mistral, version décembre 2025. Pas spécifiquement agentique mais excellente base pour des agents qui font surtout de la génération de code.

```bash
ollama pull codestral
ollama run codestral
>>> /set parameter num_ctx 32768
>>> /save codestral:32k
```

**Bon choix si** : tu veux un comportement déterministe, sans MoE, pour des tâches de codage structuré.

---

## 5. Qwen2.5-Coder 7B — léger et rapide

Le modèle d'entrée pour des **petits agents** ou tâches de complétion.

```bash
ollama pull qwen2.5-coder:7b
ollama run qwen2.5-coder:7b
>>> /set parameter num_ctx 32768
>>> /save qwen2.5-coder:7b-32k
```

**Usage** : agents simples (classification, extraction, refactor mono-fichier), pré-filtre avant un modèle plus gros.

---

## ❌ À éviter pour l'agentique

| Modèle | Pourquoi |
|---|---|
| **Llama 3.1 70B** | 42 GB → offload CPU obligatoire, ~2-5 t/s, inutilisable en boucle |
| **DeepSeek-R1** (reasoning) | Génère des `<think>` blocks longs, latence prohibitive |
| **Magistral** (reasoning Mistral) | Même problème : raisonnement long, mauvais pour des tools en série |
| **Gemma 3** | Tool calling instable, conçu pour le chat |
| **Llama 3.1 8B** généraliste | Pas optimisé code, faible sur SWE-bench |
| **Mistral 7B** classique | Surclassé par tout le reste |
| **Qwen 2.5 Coder 1.5B base** | Modèle base = pas de format instruct, pas de tools |

---

## 🎯 Décision rapide selon le scénario

| Scénario | Modèle |
|---|---|
| Refactor multi-fichiers, gros repo | **Qwen3-Coder 30B-A3B** |
| Bug fix autonome style SWE-Agent | **Devstral Small 2** |
| Boucle interactive rapide (chat + tools) | **GPT-OSS 20B** |
| Génération de code déterministe | **Codestral 25.12** |
| Petit agent ou complétion | **Qwen2.5-Coder 7B** |
| Planification one-shot complexe | **Magistral** ou **DeepSeek-R1** (hors boucle agentique) |

---

## ⚠️ Trois pièges à connaître

### 1. `num_ctx = 4096` par défaut
Ollama force 4k → **les tool calls plantent dès le second tour**. Toujours créer une variante avec `num_ctx ≥ 16384`.

### 2. Bugs tool calling Qwen3 < mars 2026
Si tu utilises `llama.cpp` ou Ollama anciens, Qwen3-Coder boucle ou produit du texte au lieu d'appeler l'outil. Mettre Ollama à jour : `curl -fsSL https://ollama.com/install.sh | sh`.

### 3. MoE ≠ dense pour le scaling
Un MoE 30B/3B utilise ~6 GB VRAM en Q4 mais le **cache KV scale sur les 30B totaux**. Pour 32k de contexte, prévoir +2-3 GB. C'est pour ça que Qwen3-Coder 30B-A3B tient confortablement en 16 GB.

---

## Configuration OpenCode recommandée

Dans `~/.config/opencode/opencode.json` (avec le plugin `opencode-models-discovery`), tous ces modèles seront listés automatiquement. Modèle par défaut suggéré :

```bash
opencode --model ollama/qwen3-coder:30b-a3b-32k
# ou
export OPENCODE_MODEL="ollama/devstral-small-2:24k"
```

---

## Sources

- [Devstral Small 2 vs Qwen3 Coder 30B — Artificial Analysis](https://artificialanalysis.ai/models/comparisons/devstral-small-2-vs-qwen3-coder-30b-a3b-instruct)
- [Qwen3-Coder How to Run Locally — Unsloth](https://unsloth.ai/docs/models/tutorials/qwen3-coder-how-to-run-locally)
- [Best Local LLMs for 16GB VRAM 2026 — LocalLLM.in](https://localllm.in/blog/best-local-llms-16gb-vram)
- [Best Local Coding Models Ranked 2026 — InsiderLLM](https://insiderllm.com/guides/best-local-coding-models-2026/)
- [Devstral Small 2 24B & Qwen3 Coder 30B — Small Coding Model Era](https://jangwook.net/en/blog/en/devstral-qwen3-coder-small-models/)
- [Best Self-Hosted LLM Leaderboard 2026 — Onyx AI](https://onyx.app/self-hosted-llm-leaderboard)
- [Best Open Source Self-Hosted LLMs for Coding 2026 — Pinggy](https://pinggy.io/blog/best_open_source_self_hosted_llms_for_coding/)
