# 🧠 docs-IA-locaux

Documentation complète pour monter un stack d'**IA agentique 100 % local** basé sur **Ollama + OpenCode**, avec des modèles spécialisés par usage.

> **Cible matérielle** : 16 GB VRAM + 128 GB RAM — Linux (Debian 14+)
> **Date** : avril 2026

---

## 🚀 À qui s'adresse ce dépôt

À toute personne qui veut :

- Faire tourner des LLM en local pour du **code agentique**, de la **traduction**, de l'**analyse**, de la **data engineering**, du **RAG**, du **fine-tuning** ou de la **vision**
- Utiliser **OpenCode** comme interface CLI avec **Ollama** comme runtime
- Avoir un classement à jour des meilleurs modèles ouverts et des pièges à éviter
- Tout faire **sans API externe, sans télémétrie, sans cloud**

---

## 📖 Par où commencer

👉 **[Index complet de la documentation](./index.md)**

Parcours rapides :

| Je veux…                                           | Commencer par                                                      |
| -------------------------------------------------- | ------------------------------------------------------------------ |
| Utiliser OpenCode avec Ollama                      | [`opencode-01-installation.md`](./opencode-01-installation.md)     |
| Choisir un modèle pour du code agentique           | [`models-01-code-agentique.md`](./models-01-code-agentique.md)     |
| Traduire mes documents FR ↔ EN                     | [`models-02-traduction.md`](./models-02-traduction.md)             |
| Résumer/analyser de longs textes                   | [`models-03-analyse-resume.md`](./models-03-analyse-resume.md)     |
| Générer du SQL / pandas / dbt                      | [`models-04-data-engineering.md`](./models-04-data-engineering.md) |
| Construire un système RAG                          | [`models-05-rag-embeddings.md`](./models-05-rag-embeddings.md)     |
| Spécialiser un modèle sur mon domaine              | [`models-06-fine-tuning.md`](./models-06-fine-tuning.md)           |
| Analyser images / PDF scannés                      | [`models-07-vision-multimodal.md`](./models-07-vision-multimodal.md) |

---

## 📦 Contenu du dépôt

| #   | Document                                                                 | Sujet                                 |
| --- | ------------------------------------------------------------------------ | ------------------------------------- |
| —   | [`index.md`](./index.md)                                                 | Index global + parcours conseillés    |
| 01  | [`opencode-01-installation.md`](./opencode-01-installation.md)           | Installation d'OpenCode               |
| 02  | [`opencode-02-configuration-ollama.md`](./opencode-02-configuration-ollama.md) | Config OpenCode + Ollama + plugin auto-discovery |
| 03  | [`models-01-code-agentique.md`](./models-01-code-agentique.md)           | Top modèles code agentique            |
| 04  | [`models-02-traduction.md`](./models-02-traduction.md)                   | Top modèles traduction                |
| 05  | [`models-03-analyse-resume.md`](./models-03-analyse-resume.md)           | Top modèles analyse / résumé          |
| 06  | [`models-04-data-engineering.md`](./models-04-data-engineering.md)       | Top modèles SQL / ETL / pandas        |
| 07  | [`models-05-rag-embeddings.md`](./models-05-rag-embeddings.md)           | Pipeline RAG complet                  |
| 08  | [`models-06-fine-tuning.md`](./models-06-fine-tuning.md)                 | Fine-tuning QLoRA local               |
| 09  | [`models-07-vision-multimodal.md`](./models-07-vision-multimodal.md)     | VLM, OCR, analyse d'images            |

---

## ⚡ Quick start

```bash
# 1. Installer Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 2. Récupérer les modèles essentiels
ollama pull hf.co/unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF:UD-Q4_K_XL  # code + data
ollama pull qwen3:30b-a3b-instruct-2507                                 # analyse
ollama pull mistral-small3.1                                            # traduction
ollama pull qwen3-embedding:8b                                          # RAG
ollama pull qwen2.5vl:7b                                                # vision

# 3. Installer OpenCode + plugin d'auto-découverte des modèles
# → voir opencode-01-installation.md et opencode-02-configuration-ollama.md
```

---

## ⚠️ Principes transverses

1. **`num_ctx = 4096` par défaut casse les tool calls** → toujours créer des variantes étendues
2. **MoE > Dense** sur 16 GB VRAM pour les gros modèles (Qwen3-Coder 30B-A3B, GPT-OSS)
3. **Température basse (0.1-0.3)** pour code, SQL, traduction, OCR
4. **Évaluation sur tes données** > benchmarks publics
5. **Fine-tuning pour le style/format**, **RAG pour le savoir factuel**
6. **Reranker** ajoute +20-40 % de précision au RAG pour ~100 ms de latence
7. **Chunking** conditionne la qualité RAG plus que le modèle d'embedding

---

## 🤝 Contributions

Le dépôt reflète une configuration personnelle et évolue au fil des tests. Les retours, corrections et suggestions sont les bienvenus via des issues ou pull requests.

## 📄 Licence

Documentation personnelle — utilisation libre pour un usage personnel ou éducatif.
