# 👁️ Classement des modèles locaux — Vision & Multi-modal

Classement des **VLM (Vision Language Models)** locaux pour analyser images, documents scannés, diagrammes, captures d'écran, tableaux et figures.

> **Cible matérielle** : 16 GB VRAM + 128 GB RAM — Linux (Debian 14+)
> **Date** : avril 2026
> **Ollama requis** : ≥ 0.18 (support multi-modal natif)

---

## 🎯 Cas d'usage couverts

Un VLM ne fait pas qu'« analyser une image » — les sous-tâches sont très différentes :

| Cas d'usage                    | Exemple concret                                  | Modèle recommandé             |
| ------------------------------ | ------------------------------------------------ | ----------------------------- |
| **OCR document**               | PDF scanné → Markdown/JSON structuré             | olmOCR-2-7B, OCRFlux-3B       |
| **Compréhension document**     | Répondre à « quelle est la date d'échéance ? »   | Qwen3-VL 8B, Qwen2.5-VL 7B    |
| **Analyse de diagramme**       | Extraire données d'un graphique                  | Qwen2.5-VL 32B, Gemma 3 27B   |
| **Capture d'écran**            | « Que dit ce message d'erreur ? »                | Qwen2.5-VL 7B, Gemma 3 4B     |
| **Multi-image**                | Comparer 3 versions d'un design                  | Qwen3-VL 8B, Pixtral 12B      |
| **Vidéo courte**               | Résumer 30 s de footage                          | Qwen2.5-VL 7B/32B             |
| **Math/formule en photo**      | Photo d'équation → LaTeX + résolution            | Phi-4-reasoning-vision, Qwen3 |
| **Raisonnement spatial**       | « L'objet A est-il à gauche de B ? »             | Qwen3-VL 8B, Gemma 3 27B      |

Deux familles de modèles se distinguent :

1. **VLM généralistes** — bons partout, un peu moins forts en OCR pur
2. **OCR spécialisés** — extraction de texte/structure de document ultra précise, faibles en raisonnement visuel

---

## 🏆 Top 5 — VLM généralistes

| Rang | Modèle                         | Taille | VRAM | DocVQA  | MMMU   | Verdict                               |
| ---- | ------------------------------ | ------ | ---- | ------- | ------ | ------------------------------------- |
| 🥇   | **Qwen3-VL 8B Instruct**       | 8B     | ~10  | 96+     | 63+    | **Le meilleur 2026**, doc + raisonn.  |
| 🥈   | **Qwen2.5-VL 7B Instruct**     | 7B     | ~8   | 95.7    | 58.6   | Référence mature et stable            |
| 🥉   | **Gemma 3 27B QAT**            | 27B    | ~14  | 93      | 62     | Top qualité 16 GB, quantisé officiel  |
| 4    | **Pixtral 12B**                | 12B    | ~11  | 90.7    | 52.5   | Excellent multi-image, Apache 2.0     |
| 5    | **Llama 3.2 Vision 11B**       | 11B    | ~10  | 88      | 50.7   | 128 K contexte, rapide                |

### 🥇 Qwen3-VL 8B Instruct — le nouveau standard

- Sortie 2026, successeur direct de Qwen2.5-VL
- Architecture retravaillée : encodeur visuel MoE, positional embeddings 3D
- **Raisonnement visuel en nette hausse** (MathVista, chart understanding)
- Contexte : 128 K tokens avec images
- Multi-image : jusqu'à 256 images dans un même prompt
- Vidéo : support natif (extraction de frames)
- Licences : Apache 2.0

```bash
# Version Ollama (si dispo)
ollama pull qwen3-vl:8b

# Alternative HuggingFace GGUF
ollama pull hf.co/unsloth/Qwen3-VL-8B-Instruct-GGUF:Q4_K_M

# Variante num_ctx étendu
cat > Modelfile <<EOF
FROM qwen3-vl:8b
PARAMETER num_ctx 32768
PARAMETER temperature 0.1
EOF
ollama create qwen3-vl-32k -f Modelfile
```

**À utiliser pour** : tout nouveau projet vision. Remplace Qwen2.5-VL sauf si tu as déjà des prompts optimisés pour ce dernier.

---

### 🥈 Qwen2.5-VL 7B Instruct — la référence stable

- Modèle sorti en 2024, **énormément d'écosystème** (llama.cpp, MLX, vLLM)
- Scores : DocVQA 95.7, MMMU 58.6, MathVista 68.2
- Contexte : 32 K (128 K en mode YaRN)
- **OCR natif de haute qualité** sur 11 langues dont le français
- Gestion des **bounding boxes** et coordonnées spatiales
- Version 32B pour qualité maximale si budget offload

```bash
ollama pull qwen2.5vl:7b
# ou pour plus de qualité (offload partiel CPU)
ollama pull qwen2.5vl:32b
```

**À utiliser pour** : projets en production, workflow bounding box/layout, compréhension de formulaires.

---

### 🥉 Gemma 3 27B QAT — qualité maximale sur 16 GB

- Version **Quantization Aware Training** officielle Google
- Tient dans **14 GB VRAM** grâce au QAT INT4 (vs 54 GB en FP16)
- Contexte : 128 K tokens
- Excellent pour **raisonnement visuel complexe** (diagrammes, maths, schémas techniques)
- Multi-modal + multilingue (>140 langues)

```bash
ollama pull gemma3:27b-it-qat
# Version plus légère
ollama pull gemma3:12b-it-qat  # ~7 GB VRAM
ollama pull gemma3:4b-it-qat   # ~3 GB VRAM, parfait pour screenshots
```

**À utiliser pour** : analyse de graphiques scientifiques, schémas techniques, documents mixtes complexes. Gemma 3 4B QAT est **imbattable** pour des captures d'écran rapides.

---

### 4️⃣ Pixtral 12B — multi-image natif, Apache 2.0

- Premier VLM de Mistral, architecture originale (pas dérivée de CLIP)
- **Très fort sur multi-image** (comparaisons, séquences)
- Licence Apache 2.0 (utilisation commerciale totale)
- Contexte : 128 K tokens
- Scores : DocVQA 90.7, MMMU 52.5

```bash
ollama pull pixtral:12b
# Version Large (offload CPU requis)
ollama pull pixtral-large:124b  # lourd, pour serveur
```

**À utiliser pour** : comparer plusieurs versions d'un design/diagramme, licence totalement libre.

---

### 5️⃣ Llama 3.2 Vision 11B — contexte 128 K, rapide

- Intégré à Ollama depuis 2024, très largement déployé
- **3× plus rapide** que Qwen2.5-VL 7B en pratique
- Qualité OCR/doc légèrement inférieure (DocVQA ~88)
- Idéal pour des traitements à haut débit

```bash
ollama pull llama3.2-vision:11b
# Version lourde (offload CPU)
ollama pull llama3.2-vision:90b
```

**À utiliser pour** : pipelines à fort volume où la latence compte plus que la précision absolue (modération, classification).

---

## 🔬 Top OCR spécialisés — extraction document de haute précision

Pour les documents (PDF scannés, factures, formulaires), les **modèles OCR dédiés** surpassent les VLM généralistes.

| Rang | Modèle                | Taille | Spécificité                                    | VRAM |
| ---- | --------------------- | ------ | ---------------------------------------------- | ---- |
| 🥇   | **olmOCR-2-7B**       | 7B     | Pipeline PDF complet, fine-tune Qwen2.5-VL     | ~8   |
| 🥈   | **OCRFlux-3B**        | 3B     | Multi-colonnes, tableaux complexes, ultra rapide | ~4   |
| 🥉   | **RolmOCR**           | 7B     | Handwriting + imprimé, bon rapport perf/taille | ~8   |
| 4    | **PaddleOCR-VL 0.9B** | 0.9B   | CPU ou très petit GPU, 92.6 score OCR          | ~2   |
| 5    | **MiniCPM-V 4.5**     | 8B     | 6× plus efficace, tableaux et équations        | ~9   |

### 🥇 olmOCR-2-7B — le pipeline PDF de référence

- Projet open source d'AllenAI, pipeline PDF complet
- Fine-tuning de **Qwen2.5-VL 7B** spécialisé documents
- Gère multi-colonnes, formules, tableaux, footnotes
- Sortie : **Markdown propre** avec structure préservée
- Licence Apache 2.0

```bash
ollama pull hf.co/allenai/olmOCR-2-7B-1025-GGUF:Q4_K_M
```

Le pipeline associé (Python) gère :
- Découpe des PDF en pages
- Rotation/correction automatique
- Agrégation des pages en un seul markdown
- Table of contents

**À utiliser pour** : convertir des archives PDF en corpus Markdown propre pour du RAG.

---

### 🥈 OCRFlux-3B — champion multi-colonnes

- 3B paramètres seulement, **très rapide**
- Spécialisé **documents à mise en page complexe** (journaux, études, rapports)
- Gère : tableaux imbriqués, légendes, renvois de bas de page
- Excellent sur factures et formulaires

```bash
ollama pull hf.co/ChatDOC/OCRFlux-3B-GGUF:Q4_K_M
```

**À utiliser pour** : pipelines industriels où chaque page compte en millisecondes.

---

### 🥉 RolmOCR — imprimé + manuscrit

- Fine-tune Qwen2-VL optimisé pour la **cohabitation imprimé/manuscrit**
- Idéal pour archives historiques, notes de cours scannées, formulaires annotés
- Bon rapport précision / taille

```bash
ollama pull hf.co/reducto/RolmOCR-GGUF:Q4_K_M
```

---

### 4️⃣ PaddleOCR-VL 0.9B — l'ultra-light

- **0.9B paramètres**, tourne sur CPU ou GPU modeste
- Score OCR 92.6 sur le benchmark OmniDocBench
- Parfait pour batch processing local sans GPU dédié

```bash
# Pas Ollama, via Python PaddlePaddle
pip install paddlepaddle paddleocr
```

---

### 5️⃣ MiniCPM-V 4.5 — efficace et polyvalent

- 8B paramètres, très bon sur équations et tableaux
- 6× plus efficace que MiniCPM-V 3.0
- Support vidéo courte

```bash
ollama pull minicpm-v:8b
```

---

## 🧠 Top raisonnement visuel — pour les tâches « intelligentes »

| Modèle                       | Taille | Point fort                                |
| ---------------------------- | ------ | ----------------------------------------- |
| **Phi-4-reasoning-vision**   | 15B    | Math/chain-of-thought sur image           |
| **InternVL 3.5**             | 8B     | Multi-modal raisonnement, dialogue image  |
| **Qwen3-VL 8B Thinking**     | 8B     | Raisonnement explicite avec `<think>`     |

Ces modèles génèrent une **chaîne de raisonnement explicite** avant la réponse, utile pour :
- Résoudre une équation photographiée
- Diagnostiquer une erreur depuis un screenshot console
- Analyser un schéma d'architecture

```bash
ollama pull hf.co/microsoft/Phi-4-reasoning-vision-GGUF:Q4_K_M
ollama pull internvl:8b-3.5
```

---

## ❌ À éviter (en 2026)

- **LLaVA 1.5/1.6** — obsolète, remplacé par Qwen2.5-VL et Gemma 3
- **BakLLaVA** — obsolète
- **CogVLM v1** — lent, remplacé par CogVLM2 ou Qwen
- **MiniCPM-V 2.x** — v4.5 est 6× plus efficace
- **GPT-4V (API)** — hors scope local, cher
- **Claude 3.5 Sonnet Vision (API)** — hors scope local
- **Qwen2-VL** (v2 sans `.5`) — remplacé par Qwen2.5-VL puis Qwen3-VL

---

## 🧭 Matrice de décision rapide

```
Tâche OCR pure (PDF → Markdown) ?
 └─ Oui → olmOCR-2-7B (pipeline complet)
          OCRFlux-3B (si latence critique)

Tâche VLM généraliste (Q/R sur image) ?
 ├─ Besoin max qualité 16 GB → Gemma 3 27B QAT
 ├─ Nouveau projet         → Qwen3-VL 8B
 ├─ Production stable      → Qwen2.5-VL 7B
 ├─ Multi-image            → Pixtral 12B
 └─ Haut débit, qualité ok → Llama 3.2 Vision 11B

Raisonnement visuel complexe (math, diagramme) ?
 └─ Phi-4-reasoning-vision ou Qwen3-VL 8B Thinking

Tout petit VLM pour screenshots rapides ?
 └─ Gemma 3 4B QAT (~3 GB VRAM)

CPU seul (pas de GPU) ?
 └─ PaddleOCR-VL 0.9B (OCR) + Gemma 3 4B QAT (chat)
```

---

## 🛠️ Pipeline type — PDF → Markdown structuré

```python
# Pipeline bout-en-bout avec olmOCR-2 via Ollama
import base64
import ollama
from pathlib import Path
from pdf2image import convert_from_path

def pdf_to_markdown(pdf_path: Path, model: str = "olmocr-2:7b") -> str:
    """Convertit un PDF en markdown via un VLM OCR local."""
    pages = convert_from_path(pdf_path, dpi=200)
    markdown_parts = []

    for i, page in enumerate(pages, 1):
        # Encode l'image en base64
        buffer = BytesIO()
        page.save(buffer, format="PNG")
        img_b64 = base64.b64encode(buffer.getvalue()).decode()

        response = ollama.chat(
            model=model,
            messages=[{
                "role": "user",
                "content": "Convertis cette page en Markdown propre. "
                           "Préserve les titres, tableaux, listes. "
                           "N'ajoute rien qui n'est pas dans la page.",
                "images": [img_b64],
            }],
            options={"temperature": 0.0, "num_ctx": 16384},
        )
        markdown_parts.append(f"\n\n## Page {i}\n\n{response['message']['content']}")

    return "\n".join(markdown_parts)

# Usage
md = pdf_to_markdown(Path("rapport.pdf"))
Path("rapport.md").write_text(md)
```

---

## 🔧 Intégration avec OpenCode

Les VLM fonctionnent dans OpenCode **via le provider Ollama** (voir [`opencode-02-configuration-ollama.md`](./opencode-02-configuration-ollama.md)). Les images peuvent être passées :

1. **En collant directement** dans le prompt (OpenCode gère l'encodage)
2. **Via un chemin de fichier** que OpenCode lit et transmet

Astuce : pour analyser un screenshot d'erreur, utilise Qwen2.5-VL 7B configuré avec `num_ctx=16384` pour laisser de la place à l'image et au texte du log.

```jsonc
// ~/.config/opencode/opencode.json — sélection par défaut
{
  "model": "ollama/qwen2.5vl:7b"
}
```

---

## ⚠️ Pièges fréquents

1. **Résolution d'image trop basse** — les VLM ont besoin d'au moins **700×700 px** pour lire du texte fiable
2. **`num_ctx` trop petit** — les images consomment beaucoup de tokens (souvent 1 000+). Toujours étendre le contexte
3. **Température > 0.3 pour l'OCR** — le modèle « invente » des caractères
4. **PDF avec texte vectoriel** → ne pas passer par un VLM, utiliser `pdfplumber` / `pymupdf` directement
5. **Formules mathématiques** → préférer Phi-4-reasoning-vision ou Qwen2.5-VL, les OCR classiques cassent le LaTeX
6. **Langues non latines** — vérifier le support (Qwen2.5-VL en gère 11, Gemma 3 >140)
7. **Vidéos longues** — pas d'usage direct, extraire N frames puis analyser en batch

---

## 📊 Benchmarks clés (rappel)

| Modèle                  | DocVQA | MMMU | ChartQA | MathVista | VRAM   |
| ----------------------- | ------ | ---- | ------- | --------- | ------ |
| Qwen3-VL 8B             | 96+    | 63+  | 88+     | 73+       | 10 GB  |
| Qwen2.5-VL 7B           | 95.7   | 58.6 | 87.3    | 68.2      | 8 GB   |
| Qwen2.5-VL 32B          | 96.4   | 70   | 89.5    | 74.8      | offload|
| Gemma 3 27B QAT         | 93     | 62   | 85      | 67        | 14 GB  |
| Gemma 3 4B QAT          | 86     | 46   | 74      | 50        | 3 GB   |
| Pixtral 12B             | 90.7   | 52.5 | 81.8    | 58        | 11 GB  |
| Llama 3.2 Vision 11B    | 88     | 50.7 | 83.4    | 51.5      | 10 GB  |
| Phi-4-reasoning-vision  | 89     | 60   | 82      | 71        | 12 GB  |

---

## 🔗 Sources

- [Qwen-VL GitHub](https://github.com/QwenLM/Qwen-VL) — doc officielle
- [Ollama vision library](https://ollama.com/library?c=vision) — liste officielle
- [OpenVLM Leaderboard](https://huggingface.co/spaces/opencompass/open_vlm_leaderboard) — classement
- [MMMU benchmark](https://mmmu-benchmark.github.io/) — évaluation généraliste
- [DocVQA](https://www.docvqa.org/) — benchmark document
- [olmOCR](https://olmocr.allen.ai/) — projet OCR AllenAI
- [Gemma 3 QAT blog](https://developers.googleblog.com/en/gemma-3-quantized-aware-trained-state-of-the-art-ai-to-consumer-gpus/) — QAT officiel
