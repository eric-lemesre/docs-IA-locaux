# Fine-tuning local — spécialiser un modèle sur 16 GB VRAM

> Cible matérielle : 16 GB VRAM + 128 GB RAM. Fine-tuning + export GGUF vers Ollama.
> Focus : LoRA / QLoRA avec Unsloth, datasets, workflow bout-en-bout.

---

## 🤔 Fine-tuning vs RAG — que choisir ?

| Critère                                        | Fine-tuning    | RAG                   |
| ---------------------------------------------- | -------------- | --------------------- |
| **Injecter de nouvelles connaissances**        | ⚠️ Fragile     | ✅ Idéal               |
| **Changer le style / ton / format**            | ✅ Idéal        | ❌ Difficile           |
| **Enseigner une tâche spécifique**             | ✅              | ⚠️ Partiel            |
| **Suivre une structure stricte** (JSON custom) | ✅              | ⚠️ Prompt engineering |
| **Données fraîches / changeantes**             | ❌              | ✅                     |
| **Coût initial**                               | 💰💰 (compute) | 💰 (setup)            |
| **Coût inférence**                             | = base         | ⬆️ (retrieval)        |

**Règle 2026** :

- **RAG** pour injecter du savoir factuel
- **Fine-tuning** pour changer le comportement (style, format, domaine)
- Les deux se **combinent** très bien (fine-tune sur ton style + RAG pour les faits)

---

## 🏆 Framework recommandé — Unsloth

**Unsloth** est devenu le standard 2026 pour le fine-tuning local. Gains mesurés :

- **~2× plus rapide** qu'un pipeline LoRA HuggingFace classique
- **70 % de VRAM en moins** que le full fine-tuning
- **50-80 % de VRAM en moins** que LoRA standard
- Checkpoints 100 % compatibles HuggingFace

### Comparaison des frameworks

| Framework         | Vitesse | VRAM    | Multi-GPU | Verdict                    |
| ----------------- | ------- | ------- | --------- | -------------------------- |
| **Unsloth**       | 🥇      | 🥇      | ❌         | 🥇 Mono-GPU (ton cas)      |
| **Axolotl**       | Lent    | OK      | ✅ FSDP2   | Multi-GPU distribué        |
| **TorchTune**     | Correct | Correct | ✅         | PyTorch-pur, plus flexible |
| **HF TRL + PEFT** | Lent    | OK      | ✅         | Contrôle maximal           |
| **LLaMA-Factory** | OK      | OK      | ✅         | UI web, débutants          |

**Benchmark** : Llama-3.1 8B, QLoRA, 2 epochs, A100 40 GB → Unsloth **3.2 h** vs Axolotl **5.8 h**.

---

## 🎯 LoRA vs QLoRA — que choisir ?

### LoRA (Low-Rank Adaptation)

- Base model en **16-bit**
- Adaptateurs en 16-bit
- Qualité légèrement supérieure
- **4× plus de VRAM** que QLoRA

### QLoRA (Quantized LoRA)

- Base model en **4-bit** (NF4 ou Dynamic)
- Adaptateurs en bf16/fp16
- **Qualité quasi-identique** à LoRA en 2026 (grâce aux Dynamic 4-bit quants)
- **Standard recommandé** pour 16 GB VRAM

### VRAM requise par taille de modèle

| Modèle              | LoRA 16-bit | QLoRA 4-bit  | Tient sur 16 GB ?     |
| ------------------- | ----------- | ------------ | --------------------- |
| 7B-8B               | 16-24 GB    | **8-12 GB**  | ✅ QLoRA               |
| 14B                 | 32-40 GB    | **16-20 GB** | ⚠️ Juste (seq<512)    |
| 24B (Devstral)      | 48+ GB      | 24 GB        | ❌                     |
| 30B MoE (3B active) | -           | **17.5 GB**  | ⚠️ Juste avec Unsloth |
| 70B                 | -           | 40+ GB       | ❌                     |

**Sweet spot 16 GB** : **QLoRA sur 7B-8B**, ou **QLoRA sur MoE 30B** avec Unsloth.

---

## 🎓 Modèles de base recommandés (16 GB VRAM)

| Modèle                | Pourquoi le fine-tuner ?                      |
| --------------------- | --------------------------------------------- |
| **Qwen 2.5 7B**       | Multilingue fort, bon point de départ général |
| **Llama 3.1 8B**      | Plus grand écosystème, doc abondante          |
| **Mistral 7B v0.3**   | Excellent en français, Apache 2.0             |
| **Qwen 2.5 Coder 7B** | Pour du code spécialisé (API privée)          |
| **Gemma 4 E4B**       | Petit, rapide, idéal pour prototyper          |
| **Phi-4**             | Reasoning fort, petit footprint               |
| **Qwen3 30B-A3B**     | MoE : grand modèle, peu de VRAM à fine-tuner  |

---

## 📂 Dataset — format et qualité

### Format recommandé (ShareGPT / ChatML)

```json
{"conversations": [
  {"role": "system", "content": "Tu es un assistant juridique français."},
  {"role": "user", "content": "Qu'est-ce qu'un bail commercial ?"},
  {"role": "assistant", "content": "Un bail commercial est..."}
]}
```

Ou format **Alpaca** (plus simple) :

```json
{"instruction": "...", "input": "...", "output": "..."}
```

### Règles de qualité

| Règle                                    | Raison                                                |
| ---------------------------------------- | ----------------------------------------------------- |
| **Minimum 300 exemples**                 | Sous ce seuil, overfitting garanti                    |
| **1000-5000 exemples** est le sweet spot | Convergence stable sans overfitting                   |
| **Diversité > quantité**                 | 1000 exemples variés > 10000 similaires               |
| **Qualité > quantité**                   | Une erreur dans les labels empoisonne l'apprentissage |
| **Balance les classes**                  | Si génération par type, pas d'exemples rares          |
| **Valide sur un holdout**                | Garder 10 % jamais vu pour l'éval                     |

### Où trouver des datasets

- **HuggingFace Datasets** : 100k+ datasets, filtrer par langue/tâche
- **Anthropic/hh-rlhf** : conversations de qualité
- **OpenHermes 2.5** : 1M d'exemples chat structurés
- **No_Robots** : 10k exemples human-written
- **ChatML français** : [`nicolasdec/french-ChatML`](https://huggingface.co/datasets)
- **Generer le tien** : avec un gros modèle (synthetic data)

### Génération de dataset synthétique

```python
# Pseudo-code : générer des paires Q/R avec un gros modèle
import ollama

def generate_training_pair(topic):
    prompt = f"""Génère une question complexe sur "{topic}"
    et sa réponse détaillée et juste. Format JSON :
    {{"question": "...", "answer": "..."}}"""
    return ollama.generate(model="qwen3:30b-a3b-instruct-2507", prompt=prompt)

topics = ["contrat", "bail", "propriété", ...]  # 100 topics
dataset = [generate_training_pair(t) for t in topics for _ in range(20)]
```

---

## 🚀 Workflow complet — Unsloth + QLoRA

### 1. Installation

```bash
# Dans un venv Python 3.11+
pip install "unsloth[colab-new] @ git+https://github.com/unslothai/unsloth.git"
pip install --no-deps xformers trl peft accelerate bitsandbytes datasets
```

### 2. Script de fine-tuning (Qwen 2.5 7B + QLoRA)

```python
from unsloth import FastLanguageModel
from trl import SFTTrainer
from transformers import TrainingArguments
from datasets import load_dataset

# Charger le modèle en 4-bit
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Qwen2.5-7B-Instruct",
    max_seq_length=4096,
    load_in_4bit=True,
)

# Configurer LoRA
model = FastLanguageModel.get_peft_model(
    model,
    r=16,                    # rang des adaptateurs
    target_modules=[         # cibler attention + MLP (recommandé)
        "q_proj", "k_proj", "v_proj", "o_proj",
        "gate_proj", "up_proj", "down_proj",
    ],
    lora_alpha=32,           # scaling = 2× le rang
    lora_dropout=0.05,
    bias="none",
    use_gradient_checkpointing="unsloth",  # économise VRAM
    random_state=42,
)

# Charger le dataset
dataset = load_dataset("json", data_files="train.jsonl", split="train")

# Fonction de formatage (adapter au format de ton dataset)
def format_prompt(example):
    return {"text": tokenizer.apply_chat_template(
        example["conversations"],
        tokenize=False,
        add_generation_prompt=False
    )}

dataset = dataset.map(format_prompt)

# Entraînement
trainer = SFTTrainer(
    model=model,
    tokenizer=tokenizer,
    train_dataset=dataset,
    dataset_text_field="text",
    max_seq_length=4096,
    args=TrainingArguments(
        per_device_train_batch_size=2,
        gradient_accumulation_steps=4,    # batch effectif = 8
        warmup_steps=10,
        num_train_epochs=3,
        learning_rate=2e-4,
        fp16=False,
        bf16=True,
        logging_steps=5,
        optim="adamw_8bit",
        weight_decay=0.01,
        lr_scheduler_type="cosine",
        seed=42,
        output_dir="./outputs",
        save_strategy="epoch",
    ),
)

trainer.train()

# Sauvegarder l'adaptateur LoRA
model.save_pretrained("qwen2.5-7b-lora-adapter")
tokenizer.save_pretrained("qwen2.5-7b-lora-adapter")
```

### 3. Monitoring pendant l'entraînement

Surveiller **3 indicateurs clés** :

| Métrique     | Sain                  | Problème                                               |
| ------------ | --------------------- | ------------------------------------------------------ |
| `train_loss` | Décroît régulièrement | Plateau → LR trop bas ; explose → LR trop haut         |
| `eval_loss`  | Décroît puis plateau  | Monte alors que `train_loss` descend → **overfitting** |
| VRAM         | 80-95 %               | 100 % → OOM imminent, réduire batch ou seq             |

### 4. Fusion et export GGUF (pour Ollama)

```python
# Après l'entraînement, fusionner LoRA dans le modèle de base
model.save_pretrained_merged(
    "qwen2.5-7b-custom",
    tokenizer,
    save_method="merged_16bit",
)

# Export GGUF quantisé pour Ollama
model.save_pretrained_gguf(
    "qwen2.5-7b-custom-gguf",
    tokenizer,
    quantization_method="q4_k_m",  # ou "q5_k_m", "q8_0"
)
```

### 5. Importer dans Ollama

```bash
# Créer un Modelfile
cat > Modelfile <<EOF
FROM ./qwen2.5-7b-custom-gguf/unsloth.Q4_K_M.gguf
TEMPLATE """{{ if .System }}<|im_start|>system
{{ .System }}<|im_end|>
{{ end }}{{ if .Prompt }}<|im_start|>user
{{ .Prompt }}<|im_end|>
<|im_start|>assistant
{{ end }}{{ .Response }}<|im_end|>
"""
PARAMETER num_ctx 4096
PARAMETER temperature 0.7
SYSTEM "Tu es l'assistant spécialisé que j'ai entraîné."
EOF

# Import
ollama create qwen2.5-7b-custom -f Modelfile

# Test
ollama run qwen2.5-7b-custom "Bonjour"
```

---

## ⚙️ Hyperparamètres — guide pratique

### Learning rate

| Cas                        | LR recommandé              |
| -------------------------- | -------------------------- |
| QLoRA standard             | **2e-4** (point de départ) |
| Modèle très grand (30B+)   | 1e-4                       |
| Dataset très petit (< 500) | 5e-5                       |
| Continued pretraining      | 5e-5 à 1e-4                |
| LR trop haut (symptôme)    | Loss explose à NaN         |
| LR trop bas (symptôme)     | Loss plafonne vite         |

### Rank (r)

| r        | Usage                                    |
| -------- | ---------------------------------------- |
| **8**    | Adaptations mineures (style, format)     |
| **16**   | ✅ Par défaut recommandé                  |
| **32**   | Tâches complexes, domaines techniques    |
| **64**   | Changement majeur de comportement (rare) |
| **128+** | Quasi = full fine-tuning, perd l'intérêt |

**Règle** : `lora_alpha = 2 × r` généralement.

### Epochs

| Dataset         | Epochs |
| --------------- | ------ |
| < 1000 exemples | 1-3    |
| 1000-10 000     | 2-5    |
| > 10 000        | 1-2    |

⚠️ Au-delà de 5 epochs sur un petit dataset → **overfitting quasi-certain**.

### Batch size effectif

```
batch_effectif = per_device_train_batch_size × gradient_accumulation_steps
```

| VRAM               | Config recommandée                        |
| ------------------ | ----------------------------------------- |
| 16 GB (7B)         | `per_device=2, grad_accum=4` → effectif 8 |
| 16 GB (7B, seq=8k) | `per_device=1, grad_accum=8` → effectif 8 |
| 16 GB (MoE 30B)    | `per_device=1, grad_accum=8` → effectif 8 |

---

## 🧪 Évaluation post fine-tuning

### 1. Évaluation automatique sur holdout

```python
# 10 % du dataset non vu à l'entraînement
eval_dataset = load_dataset("json", data_files="eval.jsonl", split="train")

from trl import SFTTrainer
trainer = SFTTrainer(..., eval_dataset=eval_dataset, ...)
metrics = trainer.evaluate()  # perplexité, eval_loss
```

### 2. Évaluation LLM-as-judge

Utiliser un modèle plus gros pour juger la qualité des réponses :

```python
# Pseudo-code
judge_prompt = f"""Question: {q}
Réponse modèle fine-tuné: {a1}
Réponse modèle base: {a2}

Laquelle est meilleure ? Note chaque sur 10 (cohérence, précision, ton).
Format JSON: {{"fine_tuned": X, "base": Y, "winner": "..."}}"""

scores = ollama.generate(model="qwen3:30b-a3b-instruct-2507", prompt=judge_prompt)
```

### 3. Benchmarks spécifiques au domaine

- **Code** : HumanEval, MBPP
- **Q&A** : SQuAD, TriviaQA
- **Traduction** : BLEU, COMET
- **Reasoning** : MMLU, GSM8K
- **Custom** : toujours construire un set de **50-100 questions maison**

---

## ❌ Erreurs fréquentes

### 1. Overfitting

**Symptôme** : `train_loss → 0`, `eval_loss ↗`.
**Fix** : réduire epochs, augmenter dropout (0.1), agrandir le dataset, early stopping.

### 2. Catastrophic forgetting

**Symptôme** : le modèle oublie ses compétences générales après fine-tuning (ex : ne sait plus coder).
**Fix** : mélanger le dataset spécifique avec 10-20 % de données généralistes (OpenHermes, etc.). Ou entraîner un LoRA séparé.

### 3. OOM (Out of Memory)

**Symptôme** : `CUDA out of memory`.
**Fix** : dans l'ordre

1. Réduire `per_device_train_batch_size` à 1
2. Augmenter `gradient_accumulation_steps`
3. Réduire `max_seq_length`
4. Activer `gradient_checkpointing="unsloth"`
5. Passer de LoRA → QLoRA (4-bit)

### 4. Format de prompt incohérent

Le template d'inférence doit **exactement** correspondre au template d'entraînement. Sinon, le modèle produit du charabia.

### 5. Dataset pollué

Un seul exemple mal labellisé peut créer un biais. Toujours **inspecter manuellement** 20-50 exemples aléatoires avant de lancer l'entraînement.

### 6. Mauvais template Ollama

Après export GGUF, si tu utilises un `TEMPLATE` Ollama qui ne correspond pas au format d'entraînement, le modèle donne des réponses cassées. **Copier le template du modèle de base**.

### 7. Tester uniquement sur des cas connus

Toujours tester sur des **cas out-of-distribution** pour détecter la régression.

---

## 💡 Cas d'usage fréquents

### 1. Assistant domaine spécifique (juridique, médical, technique)

- **Base** : Qwen 2.5 7B Instruct
- **Dataset** : 2000-5000 Q/R du domaine (du RAG peut aussi générer des paires)
- **Rank** : 16
- **Epochs** : 2-3

### 2. Générateur de code dans un framework interne

- **Base** : Qwen 2.5 Coder 7B
- **Dataset** : PRs internes, docstrings, exemples canoniques (500-3000)
- **Rank** : 32
- **Epochs** : 3
- ⚠️ Éviter de passer les secrets en clair dans le dataset

### 3. Style éditorial / voix de marque

- **Base** : Mistral 7B v0.3
- **Dataset** : 500-2000 textes écrits dans le style cible
- **Rank** : 8-16
- **Epochs** : 2
- **Note** : le style s'apprend vite, ne pas surentraîner

### 4. Traduction FR↔EN avec glossaire métier

- **Base** : Mistral Small 3.1 (si tu as 24 GB) ou Qwen 2.5 7B
- **Dataset** : paires FR↔EN du domaine (3000-10 000)
- **Rank** : 16-32
- **Epochs** : 2

### 5. Format de sortie strict (JSON custom)

- **Base** : Qwen 2.5 7B
- **Dataset** : 500-2000 paires input → JSON exact (toujours structure identique)
- **Rank** : 8 (changement mineur)
- **Epochs** : 2-3

---

## 🔄 Continued pretraining (avancé)

Fine-tuner le modèle sur du **texte brut** (pas des paires Q/R) pour injecter du savoir de domaine.

```python
# Dataset = corpus de texte brut (articles, docs internes, etc.)
# Pas de chat template, juste du texte continu
dataset = load_dataset("text", data_files="corpus.txt")
```

- **LR plus bas** : 5e-5 à 1e-4
- **Plus d'epochs** : 3-5
- **Rank plus haut** : 32-64
- **Combiner** ensuite avec un SFT sur des Q/R du domaine

---

## 🧰 Alternatives à Unsloth

### Axolotl — multi-GPU / production

Config YAML déclarative, excellent si tu scales à plusieurs GPUs :

```yaml
base_model: Qwen/Qwen2.5-7B-Instruct
load_in_4bit: true
adapter: qlora
lora_r: 16
lora_alpha: 32
datasets:
  - path: train.jsonl
    type: sharegpt
num_epochs: 3
micro_batch_size: 2
gradient_accumulation_steps: 4
learning_rate: 2e-4
```

### LLaMA-Factory — UI web

Pour qui veut fine-tuner sans écrire de code :

```bash
llamafactory-cli webui
```

### HuggingFace TRL + PEFT

Maximum de flexibilité et contrôle, mais plus verbeux.

---

## 📊 Coût/bénéfice réaliste

### Coût compute (sur 16 GB VRAM local)

| Dataset         | Modèle              | Durée (RTX 4090 equiv.) |
| --------------- | ------------------- | ----------------------- |
| 1000 exemples   | Qwen 2.5 7B QLoRA   | ~30 min                 |
| 5000 exemples   | Qwen 2.5 7B QLoRA   | ~2 h                    |
| 10 000 exemples | Qwen 2.5 7B QLoRA   | ~4 h                    |
| 5000 exemples   | Qwen3 30B-A3B QLoRA | ~6 h                    |

### Quand fine-tuner vs rester en prompt engineering

| Besoin                        | Solution                 |
| ----------------------------- | ------------------------ |
| Changer 1-2 fois la réponse   | Few-shot dans le prompt  |
| Format stable sur 1 tâche     | System prompt + examples |
| Style cohérent sur 100 tâches | **Fine-tuning**          |
| Injecter 10 docs              | RAG                      |
| Injecter 10 000 docs          | RAG                      |
| Vraie spécialisation domaine  | **Fine-tuning + RAG**    |

---

## 🎯 Récap — workflow pragmatique 16 GB VRAM

```
1. Clarifier l'objectif (style ? format ? domaine ?)
2. Construire dataset 1000-5000 exemples (qualité > quantité)
3. Split 90/10 (train/eval)
4. Unsloth + QLoRA sur Qwen 2.5 7B (r=16, 3 epochs, LR 2e-4)
5. Monitor : train_loss décroît, eval_loss ne remonte pas
6. Export GGUF Q4_K_M
7. Import Ollama avec Modelfile
8. Benchmark sur 50 questions out-of-distribution
9. Si pas assez bon : dataset plus gros/varié, pas plus d'epochs
10. Si trop rigide : r plus bas, dataset plus diversifié
```

---

## Sources

- [Unsloth — Fine-tuning LLMs Guide](https://unsloth.ai/docs/get-started/fine-tuning-llms-guide)
- [Unsloth — LoRA Hyperparameters Guide](https://unsloth.ai/docs/get-started/fine-tuning-llms-guide/lora-hyperparameters-guide)
- [Unsloth — Fine-tune gpt-oss Tutorial](https://unsloth.ai/docs/models/gpt-oss-how-to-run-and-fine-tune/tutorial-how-to-fine-tune-gpt-oss)
- [How to Fine-Tune Local LLMs in 2026 — SitePoint](https://www.sitepoint.com/fine-tune-local-llms-2026/)
- [Axolotl vs Unsloth vs TorchTune 2026 — Spheron](https://www.spheron.network/blog/axolotl-vs-unsloth-vs-torchtune/)
- [QLoRA Fine-Tuning with Unsloth — Medium](https://medium.com/@matteo28/qlora-fine-tuning-with-unsloth-a-complete-guide-8652c9c7edb3)
- [Unsloth and Training Hub — Red Hat Developer](https://developers.redhat.com/articles/2026/04/01/unsloth-and-training-hub-lightning-fast-lora-and-qlora-fine-tuning)
- [Fine-Tune Gemma 4 with LoRA & QLoRA — Lushbinary](https://lushbinary.com/blog/fine-tune-gemma-4-lora-qlora-complete-guide/)
- [Fine Tuning LLMs Complete Guide 2026 — Solguruz](https://solguruz.com/generative-ai/fine-tuning-llm/)
