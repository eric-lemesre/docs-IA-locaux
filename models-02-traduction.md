# Classement — Modèles locaux pour la **traduction de documents**

> Cible matérielle : 16 GB VRAM + 128 GB RAM. Inférence via Ollama.
> Focus : traductions longues, FR ↔ EN, cohérence terminologique, mise en forme préservée.

---

## 🏆 Top 5 — Traduction de documents sur 16 GB VRAM

| # | Modèle | Taille | Force principale | FR ↔ EN |
|---|---|---|---|---|
| 1 | **Mistral Small 3.1** | 24B | EU langues, terminologie cohérente | ⭐⭐⭐⭐⭐ |
| 2 | **Qwen 3 14B** | 14B | Multilingue le plus large | ⭐⭐⭐⭐ |
| 3 | **Llama 3.3 70B** (offload) | 70B | Qualité littéraire, gros docs | ⭐⭐⭐⭐⭐ |
| 4 | **Mistral 7B** | 7B | Rapide, FR natif | ⭐⭐⭐⭐ |
| 5 | **Stable LM 2 12B** | 12B | Léger FR/EN/DE/IT/ES/PT/NL | ⭐⭐⭐ |

---

## 1. Mistral Small 3.1 — 🥇 référence FR/EN

24B dense, **conçu en France** avec un focus explicite sur les langues européennes. Mistral est entraîné majoritairement sur du corpus FR/DE/ES/IT propre.

### Forces
- **Terminologie cohérente** sur des documents longs (essentiel en juridique/technique)
- Préserve la mise en forme (Markdown, HTML, listes, citations)
- Style naturel en FR (pas l'anglicisme typique des modèles US)
- Tient en 16 GB Q4 avec 32k de contexte

### Installation
```bash
ollama pull mistral-small3.1

ollama run mistral-small3.1
>>> /set parameter num_ctx 32768
>>> /set parameter temperature 0.2
>>> /save mistral-small3.1:trad
```

### Prompt type
```
Tu es traducteur FR↔EN professionnel. Traduis le texte ci-dessous
en préservant : la mise en forme Markdown, les noms propres, le ton,
les citations entre guillemets. Ne traduis PAS les blocs de code.

Glossaire :
- "lakehouse" → "lakehouse" (terme technique non traduit)
- "agentique" ↔ "agentic"

Texte source :
---
{texte}
```

---

## 2. Qwen 3 14B — couverture multilingue maximale

Le modèle **le plus polyvalent** pour les langues non européennes (CJK, arabe, hindi, etc.). Si tu traduis vers/depuis du chinois, japonais, coréen → c'est lui.

### Forces
- Support 100+ langues, le meilleur de sa catégorie
- Excellent même en sortant de FR↔EN (FR↔JA, EN↔ZH...)
- 128k de contexte théorique (16-32k fiable)

### Installation
```bash
ollama pull qwen3:14b
ollama run qwen3:14b
>>> /set parameter num_ctx 32768
>>> /save qwen3:14b-trad
```

### Quand le préférer à Mistral
- Documents multilingues (3+ langues)
- Paires asiatiques
- Si tu as besoin du **mode `/think`** pour traductions techniques pointues (ralentit mais améliore la précision)

---

## 3. Llama 3.3 70B — qualité littéraire (offload CPU)

70B dense → **ne tient PAS en VRAM**, mais avec tes **128 GB RAM** tu peux faire de l'offload CPU complet à ~3-8 tokens/s. Acceptable pour de la traduction batch overnight.

### Forces
- Qualité de style supérieure pour la fiction, le journalisme, les essais
- Préserve mieux les nuances stylistiques (registre, tonalité)
- Excellent en suivi de glossaire long

### Installation
```bash
ollama pull llama3.3:70b

# Forcer offload CPU partiel (ajuster selon VRAM dispo)
OLLAMA_NUM_GPU=20 ollama run llama3.3:70b
```

### Cas d'usage
- **Batch nocturne** : 50 documents à traduire pendant la nuit
- Livres, articles longs, contenus marketing
- Pas pour de l'interactif

---

## 4. Mistral 7B — rapide et efficient

Le modèle "workhorse" pour de la traduction haute vitesse en FR/EN, allemand, espagnol, italien.

### Forces
- 30+ tokens/s sur 16 GB VRAM
- Excellent rapport qualité/vitesse pour des paires européennes courantes
- Apache 2.0

### Installation
```bash
ollama pull mistral:7b
ollama run mistral:7b
>>> /set parameter num_ctx 16384
>>> /set parameter temperature 0.1
>>> /save mistral:7b-trad
```

**Usage** : pré-traduction massive (à valider/raffiner ensuite avec un modèle plus gros).

---

## 5. Stable LM 2 12B — spécialisé EU

Entraîné nativement sur **EN/ES/DE/IT/FR/PT/NL** → bon choix pour multi-langues européennes sur petit hardware.

```bash
ollama pull stablelm2:12b
```

**Usage de niche** : pipeline de traduction strictement européen, sans avoir besoin de couvrir l'asiatique.

---

## ❌ À éviter pour la traduction

| Modèle | Pourquoi |
|---|---|
| **DeepSeek-R1** | Reasoning excessif, ajoute des explications non demandées |
| **Magistral** | Idem, latence prohibitive |
| **Devstral / Qwen Coder** | Modèles code, faibles en littéraire |
| **Gemma 3** | Multilingue limité, dérive vers l'anglais |
| **GPT-OSS 20B** | Bon mais sur-entraîné EN, biais culturel US fort |
| **Phi-4** | Petit contexte (16k) → casse les longs docs |

---

## 🎯 Décision rapide

| Scénario | Modèle |
|---|---|
| Document technique FR↔EN (terminologie) | **Mistral Small 3.1** |
| Document multilingue avec CJK/Arabe | **Qwen 3 14B** |
| Roman / texte littéraire (qualité max) | **Llama 3.3 70B** (offload) |
| Pré-traduction massive en batch | **Mistral 7B** |
| Document EU multi-langues (DE/IT/ES + FR) | **Stable LM 2 12B** |
| Sous-titres / phrases courtes | **Mistral 7B** |
| Documentation logicielle (préserver code) | **Mistral Small 3.1** |

---

## 🛠️ Workflow recommandé pour gros documents

### Pipeline en 3 passes

```
1. Découper le document en chunks de 2-4k tokens (par paragraphe/section)
2. Traduire chaque chunk avec Mistral Small 3.1 (32k contexte = chunks + contexte)
3. Passe de relecture finale avec Llama 3.3 70B (overnight) si qualité critique
```

### Préserver la mise en forme

Pour des PDF/DOCX, **convertir d'abord en Markdown** (avec `pandoc` ou `marker`) → traduire le Markdown → reconvertir. Le LLM préserve mieux le Markdown que le HTML/XML.

```bash
# Conversion PDF → Markdown
marker_single document.pdf --output_dir ./md/

# Traduction
ollama run mistral-small3.1:trad < md/document.md > md/document_fr.md

# Reconversion
pandoc md/document_fr.md -o document_fr.pdf
```

### Glossaire injecté

Toujours injecter un glossaire dans le prompt système. Pour des projets récurrents, créer un Modelfile Ollama avec le glossaire intégré :

```dockerfile
# Modelfile
FROM mistral-small3.1
PARAMETER num_ctx 32768
PARAMETER temperature 0.2
SYSTEM """
Tu es traducteur FR↔EN. Glossaire obligatoire :
- "agent" → "agent" (pas "agente")
- "framework" → "framework" (pas "cadriciel")
- "embedding" → "embedding" (pas "plongement")
- ...
"""
```

```bash
ollama create mistral-trad-projet -f Modelfile
```

---

## ⚠️ Pièges spécifiques à la traduction

### 1. Hallucinations de noms propres
Les LLM peuvent traduire involontairement des noms propres ("New York" → "Nouvelle-York"). **Toujours préciser** dans le prompt : *« ne traduis pas les noms propres »*.

### 2. Code et URLs
Sans instruction explicite, le modèle peut "traduire" des variables ou URLs. Préciser : *« préserve le code, les URLs, les noms de fichiers tels quels »*.

### 3. Genre grammatical
EN → FR : "the developer" peut donner "le développeur" / "la développeuse". Pour la cohérence, fixer dans le prompt : *« utilise le masculin générique pour les rôles génériques »* (ou féminin/inclusif selon préférence).

### 4. Citations
Les guillemets EN (`""`) deviennent FR (`«»`) automatiquement avec Mistral. Si tu veux préserver, le préciser.

### 5. Température
Pour la traduction, **toujours basse** (0.1 - 0.3). Au-dessus, le modèle "réécrit" au lieu de traduire.

---

## Configuration OpenCode pour traduction

```bash
# Modèle par défaut traduction
opencode --model ollama/mistral-small3.1:trad "traduis ce fichier README.md en anglais"

# Ou pour usage interactif
opencode
# puis /models → ollama/mistral-small3.1:trad
```

---

## Sources

- [Best LLM for Translation 2026 — NoviAI](https://www.noviai.ai/models-prompts/best-llm-for-translation/)
- [Best LLM for Translation 2026 — Lokalise](https://lokalise.com/blog/what-is-the-best-llm-for-translation/)
- [8 Best LLMs for Translation 2026 — Vozo AI](https://www.vozo.ai/blogs/8-best-llms-for-translation-2026)
- [Top 20 Multilingual AI Models 2025 — Local AI Zone](https://local-ai-zone.github.io/guides/best-ai-multilingual-models-ultimate-ranking-2025.html)
- [10 Best Multilingual LLMs for 2026 — Azumo](https://azumo.com/artificial-intelligence/ai-insights/multilingual-llms)
- [Best LLM for Translation — Hakuna Matata Tech](https://www.hakunamatatatech.com/our-resources/blog/best-llm-for-translation)
- [Stable LM 2 — Ollama Library](https://ollama.com/library/stablelm2)
