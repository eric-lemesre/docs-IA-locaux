# Classement — Modèles locaux pour **analyse et résumé de texte**

> Cible matérielle : 16 GB VRAM + 128 GB RAM. Inférence via Ollama.
> Focus : résumés fidèles, analyse argumentative, extraction d'informations, long contexte.

---

## 🏆 Top 5 — Analyse / résumé sur 16 GB VRAM

| # | Modèle | Taille | Active | Contexte fiable | Verdict |
|---|---|---|---|---|---|
| 1 | **Qwen3 30B-A3B Instruct 2507** | 30B MoE | 3B | 32k | 🥇 Meilleur global |
| 2 | **Llama 3.3 8B** | 8B | 8B | 16-32k | 🥈 Polyvalent rapide |
| 3 | **DeepSeek-R1** (14B distill) | 14B | 14B | 16k | 🥉 Analyse profonde |
| 4 | **Mistral Small 3.1** | 24B | 24B | 32k | Synthèse structurée |
| 5 | **Phi-4** | 14B | 14B | 16k | Reasoning par GB |

> ⚠️ **Vérité du contexte long** : tous les modèles annoncent 128k tokens, mais la **qualité réelle dégrade après 16-32k**. Voir section dédiée plus bas.

---

## 1. Qwen3 30B-A3B Instruct 2507 — 🥇 le meilleur

MoE 30B (3B actifs) version juillet 2025, classé **#1 open-source pour le résumé** dans les benchmarks 2026.

### Forces
- Exceptionnel sur la **compréhension de texte** (MMLU, ARC, HellaSwag)
- Tient **40k tokens utiles** en 16 GB VRAM grâce au MoE
- Résumés fidèles, peu de paraphrase inventive
- Excellent en extraction structurée (JSON, tableaux)

### Installation
```bash
ollama pull qwen3:30b-a3b-instruct-2507

ollama run qwen3:30b-a3b-instruct-2507
>>> /set parameter num_ctx 32768
>>> /set parameter temperature 0.3
>>> /save qwen3:30b-a3b-resume
```

### Prompt type pour résumé
```
Résume le texte ci-dessous en respectant strictement :
1. Maximum 5 puces principales
2. Conserve les chiffres et noms exacts
3. Pas d'opinion personnelle
4. Indique entre crochets [§N] le numéro de paragraphe source

Texte :
---
{texte}
```

---

## 2. Llama 3.3 8B — 🥈 polyvalent rapide

Le modèle "couteau suisse" recommandé en 2026 pour les usages quotidiens.

### Forces
- 30-40 tokens/s sur 16 GB VRAM
- Bon résumé général, suivi d'instructions correct
- Apache 2.0
- Parfait pour des **pipelines de traitement par batch**

### Installation
```bash
ollama pull llama3.3:8b
ollama run llama3.3:8b
>>> /set parameter num_ctx 32768
>>> /save llama3.3:8b-32k
```

**Usage** : résumé d'emails, articles de blog, comptes-rendus de réunion. Volume > qualité littéraire.

---

## 3. DeepSeek-R1 (distill 14B) — 🥉 analyse profonde

Reasoning model qui **explicite sa chaîne de pensée**. Plus lent mais **meilleur pour l'analyse argumentative** (critique, comparaison, cohérence logique).

### Forces
- Identifie les contradictions internes d'un texte
- Excellent pour **fact-checking** automatisé
- Analyse multi-critères (pondération, comparaison)

### Installation
```bash
ollama pull deepseek-r1:14b

ollama run deepseek-r1:14b
>>> /set parameter num_ctx 16384
>>> /save deepseek-r1:14b-analyse
```

### Quand l'utiliser
- Analyse critique d'un argumentaire
- Détection de biais, sophismes
- Comparaison entre plusieurs documents
- ❌ **Pas pour du résumé simple** : trop verbeux

### Limitations
- Génère des `<think>` blocks longs (peut doubler le temps de réponse)
- Contexte fiable plus court (16k vs 32k)

---

## 4. Mistral Small 3.1 — synthèse structurée

24B dense français, excellent pour des **comptes-rendus structurés** (sections, sous-sections, plan).

### Forces
- Préserve la structure du texte source (titres, hiérarchie)
- Style synthétique élégant en français
- Tient 32k contexte fiable en Q4

### Installation
```bash
ollama pull mistral-small3.1
ollama run mistral-small3.1
>>> /set parameter num_ctx 32768
>>> /save mistral-small3.1:resume
```

**Usage** : minutes de réunion, synthèses de rapports, fiches de lecture.

---

## 5. Phi-4 — meilleur reasoning par GB

14B dense par Microsoft, **80.4 % sur MATH benchmark** (vs 68 % Llama 3.3 8B).

### Forces
- Top reasoning à taille égale
- Idéal pour **analyse quantitative** (chiffres, statistiques, graphiques décrits)
- Léger (~9 GB en Q4)

### Installation
```bash
ollama pull phi4
ollama run phi4
>>> /set parameter num_ctx 16384
>>> /save phi4:analyse
```

### Limitation majeure
- **Contexte 16k seulement** → inutilisable pour des documents longs
- À combiner avec du chunking pour de longs textes

---

## ❌ À éviter pour analyse/résumé

| Modèle | Pourquoi |
|---|---|
| **Devstral / Qwen-Coder** | Modèles code, faibles en littéraire |
| **Llama 3.1 70B** | Trop lent en offload pour de l'interactif |
| **Gemma 3** | Tendance à inventer des détails (hallucinations) |
| **Mistral 7B classique** | Surclassé par Mistral Small 3.1 |
| **Magistral** | Reasoning verbeux non désiré pour du résumé |
| **GPT-OSS 20B** | Excellent mais biais EN-centric, moins bon en FR analytique |

---

## 🎯 Décision rapide selon la tâche

| Tâche | Modèle |
|---|---|
| Résumé fidèle d'article (FR/EN) | **Qwen3 30B-A3B Instruct** |
| Résumé batch (volume) | **Llama 3.3 8B** |
| Analyse critique / argumentation | **DeepSeek-R1 14B** |
| Compte-rendu structuré (FR) | **Mistral Small 3.1** |
| Analyse de chiffres / quantitatif | **Phi-4** |
| Extraction JSON / données structurées | **Qwen3 30B-A3B Instruct** |
| Comparaison multi-documents | **DeepSeek-R1 14B** |
| Fiches de lecture | **Mistral Small 3.1** |

---

## ⚠️ Le mythe du contexte long

### Annoncé vs réel

| Modèle | Contexte annoncé | Contexte **fiable** |
|---|---|---|
| Llama 3.3 | 128k | 16-32k |
| Qwen 3 | 128k | 32k |
| Mistral Small 3.1 | 128k | 32k |
| Phi-4 | 16k | 16k |
| DeepSeek-R1 | 64k | 16k |

### Le problème "Lost in the Middle"

Les LLM **récupèrent fiablement** l'information du **début** et de la **fin** du contexte, mais **manquent** souvent les détails du **milieu**.

Pour un contexte de 128k, l'info placée entre 40k et 80k tokens est **statistiquement ignorée**.

### Solutions pratiques

1. **Placer l'info critique au début** du prompt
2. **Utiliser RAG** pour ne sortir que les chunks pertinents
3. **Chunking par sections** de 16-32k tokens avec overlap
4. **Résumé hiérarchique** : résumé par chapitre → résumé des résumés

---

## 🛠️ Workflows recommandés

### Document court (< 16k tokens)
**Direct** avec Qwen3 30B-A3B Instruct ou Mistral Small 3.1.

### Document moyen (16k - 100k tokens)
**Map-reduce** :
```
1. Découper en chunks de 8k tokens (overlap 500 tokens)
2. Résumer chaque chunk → mini-résumés (~500 tokens chacun)
3. Concaténer les mini-résumés (10-20k tokens)
4. Résumer le tout en passe finale
```

Modèle recommandé : Llama 3.3 8B pour les passes (rapide), Qwen3 30B-A3B pour la passe finale (qualité).

### Document long (livre, 100k+ tokens)
**Pipeline RAG + résumé hiérarchique** :
```
1. Indexer le document (Qdrant/Chroma + nomic-embed-text)
2. Pour chaque chapitre : récupérer chunks pertinents → résumer
3. Synthèse globale à partir des résumés de chapitres
```

### Analyse multi-documents
```
1. Embedder chaque document avec nomic-embed-text
2. Identifier les passages similaires/contradictoires (cosine sim)
3. Soumettre paires/triplets à DeepSeek-R1 pour analyse comparée
```

---

## 🔧 Modelfile prêt à l'emploi

### Résumé fidèle (zéro hallucination)
```dockerfile
FROM qwen3:30b-a3b-instruct-2507
PARAMETER num_ctx 32768
PARAMETER temperature 0.1
PARAMETER top_p 0.9
SYSTEM """
Tu es analyste documentaire. Règles strictes :
1. Tu n'inventes JAMAIS d'information absente du texte source
2. Tu cites systématiquement les passages clés entre guillemets
3. Tu indiques [SOURCE: §N] pour chaque affirmation
4. Si une information est ambiguë, tu le précises
5. Tu réponds en français
"""
```

```bash
ollama create resume-fidele -f Modelfile
```

### Analyse critique
```dockerfile
FROM deepseek-r1:14b
PARAMETER num_ctx 16384
PARAMETER temperature 0.3
SYSTEM """
Tu es analyste critique. Pour chaque texte :
1. Identifie la thèse principale
2. Liste les arguments avec leur force (faible/moyen/fort)
3. Identifie biais, sophismes, contradictions
4. Évalue la qualité des sources citées
5. Conclusion synthétique en 5 lignes max
"""
```

---

## Configuration OpenCode

Modèles à privilégier dans OpenCode pour ces usages :

```bash
# Résumé général
opencode --model ollama/qwen3:30b-a3b-resume

# Analyse profonde
opencode --model ollama/deepseek-r1:14b-analyse

# Synthèse structurée FR
opencode --model ollama/mistral-small3.1:resume
```

---

## Sources

- [Best Open Source LLMs for Summarization 2026 — SiliconFlow](https://www.siliconflow.com/articles/en/best-open-source-llms-for-summarization)
- [Long Context Local LLMs 2026 — PromptQuorum](https://www.promptquorum.com/local-llms/long-context-local-llms)
- [Best Local LLM Models 2026: Benchmarks & Use Cases — AI Tool Discovery](https://www.aitooldiscovery.com/how-to/best-local-llm-models)
- [Best LLM for Long Context — LLM Stats](https://llm-stats.com/benchmarks/category/long_context)
- [Best LLMs for Summarization 2026 — ClickUp](https://clickup.com/blog/best-llms-for-language-summarization/)
- [LLM Benchmarks 2026 — LLM Stats](https://llm-stats.com/benchmarks)
- [Best LLMs 2026: Complete Ranking — Cristhian Villegas](https://blog.iamcristhian.dev/2026/04/best-llm-2026-rankings-benchmarks-comparison)
