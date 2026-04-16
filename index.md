# 📚 Index — IA Agentique Locale

Documentation complète pour monter un stack d'IA agentique 100 % local basé sur **Ollama + OpenCode**, avec des modèles spécialisés par usage.

> **Cible matérielle** : 16 GB VRAM + 128 GB RAM — Linux (Debian 14+)
> **Date** : avril 2026

---

## 🛠️ Mise en place — OpenCode

### [`opencode-01-installation.md`](./opencode-01-installation.md)

Installation du binaire OpenCode, prérequis (bun, Ollama), localisation de l'exécutable, gestion des installations multiples, symlinks et dépannage.

### [`opencode-02-configuration-ollama.md`](./opencode-02-configuration-ollama.md)

Configuration d'OpenCode pour utiliser Ollama comme provider local, installation du plugin `opencode-models-discovery` pour la découverte automatique des modèles, gestion de `num_ctx`, multi-providers.

---

## 🏆 Classement des modèles locaux par usage

### [`models-01-code-agentique.md`](./models-01-code-agentique.md)

Classement pour le **code agentique** (boucle tools + actions + décisions).
Top 3 : **Qwen3-Coder 30B-A3B**, **Devstral Small 2**, **GPT-OSS 20B**.
Scénarios : refactor multi-fichiers, bug fix autonome, boucle interactive.

### [`models-02-traduction.md`](./models-02-traduction.md)

Classement pour la **traduction de documents** (FR↔EN, multilingue).
Top 3 : **Mistral Small 3.1**, **Qwen 3 14B**, **Llama 3.3 70B** (offload).
Workflows pour PDF/Markdown, glossaires, préservation de mise en forme.

### [`models-03-analyse-resume.md`](./models-03-analyse-resume.md)

Classement pour l'**analyse et le résumé de texte** (contexte long, extraction).
Top 3 : **Qwen3 30B-A3B Instruct 2507**, **Llama 3.3 8B**, **DeepSeek-R1 14B**.
Stratégies map-reduce, pipelines pour livres, chunking hiérarchique.

### [`models-04-data-engineering.md`](./models-04-data-engineering.md)

Classement pour la **data engineering** (SQL, ETL, pandas, dbt, Spark).
Top 3 : **Qwen3-Coder 30B-A3B**, **DeepSeek-Coder V2 16B**, **Qwen2.5-Coder 14B**.
Text-to-SQL fiable, pipelines dbt, migrations Alembic, erreurs SQL fréquentes.

### [`models-05-rag-embeddings.md`](./models-05-rag-embeddings.md)

Pipeline **RAG complet** : embeddings, reranking, vector stores, chunking.
Top embedders : **Qwen3-Embedding 8B**, **BGE-M3**, **mxbai-embed-large**.
Top rerankers : **BGE-reranker-v2-m3**, **Jina v3**.
Hybrid search, évaluation RAGAS, pipeline Python bout-en-bout.

### [`models-06-fine-tuning.md`](./models-06-fine-tuning.md)

**Fine-tuning local** avec Unsloth + QLoRA sur 16 GB VRAM.
Workflow complet : dataset → entraînement → export GGUF → import Ollama.
Hyperparamètres, overfitting, cas d'usage, coûts réalistes.

### [`models-07-vision-multimodal.md`](./models-07-vision-multimodal.md)

Classement pour la **vision et le multi-modal** (VLM, OCR, analyse de documents/images).
Top VLM : **Qwen3-VL 8B**, **Qwen2.5-VL 7B**, **Gemma 3 27B QAT**.
Top OCR : **olmOCR-2-7B**, **OCRFlux-3B**. Pipelines PDF→Markdown, multi-image.

---

## 🗺️ Parcours conseillés

### Je veux juste utiliser OpenCode avec mes modèles Ollama

1. [`opencode-01-installation.md`](./opencode-01-installation.md)
2. [`opencode-02-configuration-ollama.md`](./opencode-02-configuration-ollama.md)
3. [`models-01-code-agentique.md`](./models-01-code-agentique.md) pour choisir le bon modèle

### Je veux construire un système RAG sur ma base documentaire

1. [`models-05-rag-embeddings.md`](./models-05-rag-embeddings.md) — pipeline complet
2. [`models-03-analyse-resume.md`](./models-03-analyse-resume.md) — LLM génératif pour la synthèse
3. (optionnel) [`models-06-fine-tuning.md`](./models-06-fine-tuning.md) — fine-tune le LLM sur ton style

### Je veux un assistant data engineering

1. [`models-04-data-engineering.md`](./models-04-data-engineering.md) — choix du modèle
2. [`opencode-02-configuration-ollama.md`](./opencode-02-configuration-ollama.md) — integrer dans OpenCode
3. (optionnel) [`models-06-fine-tuning.md`](./models-06-fine-tuning.md) — fine-tune sur ton schéma

### Je veux spécialiser un modèle sur mon domaine

1. [`models-06-fine-tuning.md`](./models-06-fine-tuning.md) — workflow complet
2. [`models-05-rag-embeddings.md`](./models-05-rag-embeddings.md) — combiner avec du RAG

### Je veux analyser des images ou des documents scannés

1. [`models-07-vision-multimodal.md`](./models-07-vision-multimodal.md) — choix VLM/OCR selon le cas
2. [`opencode-02-configuration-ollama.md`](./opencode-02-configuration-ollama.md) — configurer dans OpenCode
3. (optionnel) [`models-05-rag-embeddings.md`](./models-05-rag-embeddings.md) — indexer les documents extraits

---

## 📦 Stack recommandée complète

```
┌─────────────────────────────────────────────────────┐
│            Interface : OpenCode CLI                 │
│  (avec plugin opencode-models-discovery pour auto-  │
│   découverte des modèles Ollama)                    │
└─────────────────────┬───────────────────────────────┘
                      │
        ┌─────────────┴────────────┐
        │        Ollama            │
        │   (runtime local)        │
        └─────────────┬────────────┘
                      │
    ┌─────────────────┼──────────┬──────────┐
    │                 │          │          │
    ▼                 ▼          ▼          ▼
┌─────────┐      ┌──────────┐ ┌───────┐ ┌──────────┐
│  Code   │      │  Analyse │ │  RAG  │ │  Vision  │
│Qwen3-   │      │Qwen3 30B │ │Qdrant+│ │Qwen2.5-VL│
│Coder 30B│      │A3B Inst. │ │BGE-M3 │ │  / Gemma │
│A3B      │      │          │ │bge-rk │ │  3 27B   │
└─────────┘      └──────────┘ └───────┘ └──────────┘

┌─────────────────────────────────────────────────────┐
│   Fine-tuning (Unsloth + QLoRA, hors ligne)         │
│   → export GGUF → import Ollama                     │
└─────────────────────────────────────────────────────┘
```

---

## 🎯 Cheatsheet — modèles essentiels

À avoir dans ton Ollama pour couvrir les 6 cas d'usage :

```bash
# Code agentique + data engineering (le même modèle gère les deux)
ollama pull hf.co/unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF:UD-Q4_K_XL

# Analyse et résumé
ollama pull qwen3:30b-a3b-instruct-2507

# Traduction FR/EN
ollama pull mistral-small3.1

# RAG — embeddings
ollama pull qwen3-embedding:8b
# (reranker BGE-reranker-v2-m3 via FlagEmbedding, pas Ollama)

# Petit modèle rapide polyvalent
ollama pull qwen2.5-coder:7b

# Vision / multi-modal
ollama pull qwen2.5vl:7b
# (pour l'OCR pur : olmOCR-2 ou OCRFlux-3B, voir models-07)
```

Puis créer les variantes avec `num_ctx` étendu pour chacun (voir [`opencode-02`](./opencode-02-configuration-ollama.md) §6).

---

## 📋 Vue d'ensemble des documents

| #   | Document                         | Lignes | Sujet principal                   |
| --- | -------------------------------- | ------ | --------------------------------- |
| 01  | opencode-01-installation         | 183    | Install binaire OpenCode          |
| 02  | opencode-02-configuration-ollama | 309    | Config OpenCode + Ollama + plugin |
| 03  | models-01-code-agentique         | 193    | Top modèles pour agents code      |
| 04  | models-02-traduction             | 255    | Top modèles traduction docs       |
| 05  | models-03-analyse-resume         | 305    | Top modèles analyse/résumé        |
| 06  | models-04-data-engineering       | 337    | Top modèles SQL/ETL/pandas        |
| 07  | models-05-rag-embeddings         | 539    | Pipeline RAG complet              |
| 08  | models-06-fine-tuning            | ~540   | Fine-tuning local QLoRA           |
| 09  | models-07-vision-multimodal      | ~300   | VLM, OCR, analyse d'images        |

---

## 🔗 Ressources externes principales

### Plateformes

- [Ollama Library](https://ollama.com/library) — catalogue des modèles
- [HuggingFace Models](https://huggingface.co/models) — modèles et GGUF
- [Models.dev](https://models.dev/) — catalogue utilisé par OpenCode

### Classements et benchmarks

- [MTEB Leaderboard](https://huggingface.co/spaces/mteb/leaderboard) — embeddings
- [LLM Stats](https://llm-stats.com/benchmarks) — benchmarks généraux
- [Artificial Analysis](https://artificialanalysis.ai) — comparaisons détaillées
- [Onyx AI Leaderboard](https://onyx.app/self-hosted-llm-leaderboard) — self-hosted

### Documentation officielle

- [OpenCode Docs](https://opencode.ai/docs/) — configuration complète
- [Ollama Docs](https://docs.ollama.com) — intégrations et API
- [Unsloth Docs](https://unsloth.ai/docs) — fine-tuning

---

## ⚠️ Principes transverses à tous les documents

1. **`num_ctx = 4096` par défaut casse les tool calls** → toujours créer des variantes étendues
2. **MoE (Qwen3-Coder 30B-A3B, GPT-OSS) > Dense** sur 16 GB VRAM pour les gros modèles
3. **Température basse (0.1-0.3)** pour les tâches structurées (code, SQL, traduction)
4. **Évaluation sur tes données** > benchmarks publics
5. **Fine-tuning pour le style/format**, **RAG pour le savoir factuel**
6. **Reranker ajoute +20-40 % de précision** au RAG pour ~100 ms de latence
7. **Chunking conditionne plus la qualité du RAG** que le modèle d'embedding
