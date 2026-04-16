# Classement — Modèles locaux pour la **data engineering**

> Cible matérielle : 16 GB VRAM + 128 GB RAM. Inférence via Ollama.
> Focus : SQL, ETL, pandas/polars, dbt, Spark, génération de schémas, migration de données.

---

## 🏆 Top 5 — Data engineering sur 16 GB VRAM

| #   | Modèle                    | Taille       | SQL   | Pandas/Polars | dbt/Schémas |
| --- | ------------------------- | ------------ | ----- | ------------- | ----------- |
| 1   | **Qwen3-Coder 30B-A3B**   | 30B MoE / 3B | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐         | ⭐⭐⭐⭐⭐       |
| 2   | **DeepSeek-Coder V2 16B** | 16B MoE      | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐         | ⭐⭐⭐⭐        |
| 3   | **Qwen2.5-Coder 14B**     | 14B          | ⭐⭐⭐⭐  | ⭐⭐⭐⭐          | ⭐⭐⭐⭐        |
| 4   | **GLM-4.7 Thinking**      | 9B/32B       | ⭐⭐⭐⭐  | ⭐⭐⭐⭐          | ⭐⭐⭐⭐⭐       |
| 5   | **Codestral 25.12**       | 22B          | ⭐⭐⭐⭐  | ⭐⭐⭐           | ⭐⭐⭐         |

---

## 1. Qwen3-Coder 30B-A3B — 🥇 le meilleur tout-terrain

Déjà champion en code agentique, **Qwen3-Coder est aussi #1 pour la data engineering** : génération SQL complexe, dataframes, pipelines.

### Forces data

- **Text-to-SQL** : excellent sur joins multi-tables, CTEs, fenêtres
- Maîtrise PostgreSQL, MySQL, Snowflake, BigQuery, DuckDB
- Génération **dbt models** avec `ref()`, `source()`, tests, docs
- Pandas + Polars idiomatique (suit les bonnes pratiques 2026)

### Installation

```bash
ollama pull hf.co/unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF:UD-Q4_K_XL

ollama run hf.co/unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF:UD-Q4_K_XL
>>> /set parameter num_ctx 32768
>>> /set parameter temperature 0.1
>>> /save qwen3-coder:30b-data
```

### Prompt type SQL

```
Schéma :
- orders(id, user_id, total, created_at, status)
- users(id, email, country, created_at)
- products(id, name, category, price)
- order_items(order_id, product_id, quantity, price)

Question : top 10 clients par revenu sur les 30 derniers jours,
avec leur pays et le nombre de commandes. Postgres syntax.

Renvoie uniquement le SQL, sans explication.
```

---

## 2. DeepSeek-Coder V2 16B — 🥈 spécialiste data

MoE 16B de DeepSeek, **explicitement entraîné sur Python data stack**. Souvent **supérieur à Qwen pour pandas/polars purs**.

### Forces

- Idiomes pandas/polars propres (vectorisation, méthodes chainées)
- Excellent sur Spark/PySpark
- Connaît bien Airflow, Dagster, Prefect
- Génère du code **bug-free** sur dataframes (benchmark spécifique)

### Installation

```bash
ollama pull deepseek-coder-v2:16b

ollama run deepseek-coder-v2:16b
>>> /set parameter num_ctx 32768
>>> /set parameter temperature 0.1
>>> /save deepseek-coder-v2:16b-data
```

### Quand le préférer à Qwen3-Coder

- Tâches **purement Python data** (pandas, polars, numpy, scikit, pyspark)
- Génération de notebooks Jupyter
- Code de transformation Spark complexe

---

## 3. Qwen2.5-Coder 14B — équilibre solide

Plus petit que le 30B-A3B mais **complètement dense** → comportement plus prévisible, moins de surprises.

### Forces

- Stable et déterministe
- Tient en 9 GB Q4, contexte 32k confortable
- Bon trade-off vitesse/qualité
- Excellent pour **migration de données** (script ad-hoc)

### Installation

```bash
ollama pull qwen2.5-coder:14b
ollama run qwen2.5-coder:14b
>>> /set parameter num_ctx 32768
>>> /save qwen2.5-coder:14b-data
```

**Usage** : scripts ETL one-shot, parsing CSV/JSON/XML, génération de seeds dbt.

---

## 4. GLM-4.7 Thinking — pour ETL complexes

Modèle reasoning de Zhipu AI, **excellent pour le raisonnement multi-étapes** (planifier un pipeline ETL complet).

### Forces

- Capable de **planifier des DAG Airflow/Dagster** complets
- Excellent pour le **schema design** (3NF, star schema, data vault)
- Génère des **migrations Alembic / Flyway** cohérentes
- Comprend les contraintes (FK, indexes, partitioning)

### Installation

```bash
ollama pull glm-4.7

ollama run glm-4.7
>>> /set parameter num_ctx 32768
>>> /save glm-4.7:data
```

### Quand l'utiliser

- Conception de schéma de données
- Planification d'un pipeline complet (ingestion → staging → marts)
- Migration entre stacks (ex : Postgres → Snowflake)
- ❌ Pas pour des one-liners (trop verbeux)

---

## 5. Codestral 25.12 — fallback dense

22B dense Mistral. Solide pour SQL classique mais moins fort que Qwen/DeepSeek sur les datasets modernes.

```bash
ollama pull codestral
```

**Usage** : si tu veux un modèle français/européen pour des projets soumis à des contraintes de souveraineté.

---

## ❌ À éviter pour data engineering

| Modèle                    | Pourquoi                                          |
| ------------------------- | ------------------------------------------------- |
| **Devstral**              | Optimisé bug-fixing, pas génération data          |
| **Llama 3.x généraliste** | Faible sur SQL complexe (échoue sur les fenêtres) |
| **Gemma 3**               | Pas de support dataframe propre                   |
| **Mistral 7B classique**  | Surclassé par Codestral sur tout                  |
| **DeepSeek-R1**           | Reasoning trop long pour du SQL itératif          |
| **GPT-OSS 20B**           | Bon mais Qwen-Coder spécialisé bat sur SQL        |

---

## 🎯 Décision rapide selon la tâche

| Tâche                         | Modèle                    |
| ----------------------------- | ------------------------- |
| Text-to-SQL Postgres/MySQL    | **Qwen3-Coder 30B-A3B**   |
| Pipeline pandas/polars        | **DeepSeek-Coder V2 16B** |
| Spark / PySpark               | **DeepSeek-Coder V2 16B** |
| dbt models + tests            | **Qwen3-Coder 30B-A3B**   |
| Schéma DB / migrations        | **GLM-4.7 Thinking**      |
| Script ETL ad-hoc             | **Qwen2.5-Coder 14B**     |
| Parsing CSV/JSON/XML          | **Qwen2.5-Coder 14B**     |
| Optimisation requête SQL      | **Qwen3-Coder 30B-A3B**   |
| Génération seeds / fixtures   | **Qwen2.5-Coder 14B**     |
| Notebook Jupyter exploratoire | **DeepSeek-Coder V2 16B** |

---

## ⚠️ Erreurs SQL fréquentes des LLM

Identifiées par les benchmarks Tinybird et AImultiple sur le text-to-SQL local :

| Type d'erreur                                  | Fréquence | Mitigation                                  |
| ---------------------------------------------- | --------- | ------------------------------------------- |
| **Joins incorrects** (mauvaise FK)             | ~30 %     | Toujours fournir le schéma complet avec FK  |
| **Agrégations buggées** (SUM au lieu de COUNT) | ~20 %     | Préciser explicitement la métrique attendue |
| **Filtres manquants** (NULL, dates)            | ~15 %     | Lister les filtres business attendus        |
| **Erreurs syntaxe dialecte** (PG vs MySQL)     | ~10 %     | Toujours préciser le dialecte cible         |
| **Hallucination de colonnes**                  | ~10 %     | Schéma exhaustif + temperature ≤ 0.1        |

**Règle d'or** : valider le SQL généré avec `EXPLAIN` avant de l'exécuter.

---

## 🛠️ Workflows recommandés

### Pipeline text-to-SQL fiable

```python
# Pseudo-code
schema = extract_schema_from_db()  # tables + colonnes + FK
question = "top clients par CA"

prompt = f"""
Dialecte: PostgreSQL 16
Schéma:
{schema}

Question: {question}

Renvoie uniquement la requête SQL valide. Pas d'explication.
"""

sql = ollama.generate(model="qwen3-coder:30b-data", prompt=prompt)

# Validation avant exécution
explain_result = db.execute(f"EXPLAIN {sql}")
if explain_result.is_valid:
    db.execute(sql)
```

### Pipeline dbt assisté

```bash
# 1. Générer le model
opencode --model ollama/qwen3-coder:30b-data \
  "génère un dbt model staging pour la table raw_orders avec
   - cleaning des dates
   - enum des statuts
   - tests : not_null sur id, unique sur id, accepted_values sur status"

# 2. Générer les tests
opencode --model ollama/qwen3-coder:30b-data \
  "génère le schema.yml dbt avec tests + docs pour le model stg_orders"

# 3. Run
dbt run -s stg_orders && dbt test -s stg_orders
```

### Migration de schéma

```bash
opencode --model ollama/glm-4.7:data \
  "génère une migration Alembic pour :
   - ajouter colonne 'archived_at' (timestamp nullable) à orders
   - créer index partiel sur archived_at IS NULL
   - backfill : archived_at = NULL pour toutes les lignes existantes
   - migration descendante incluse"
```

---

## 🔧 Modelfiles prêts à l'emploi

### SQL strict (zéro halluci colonne)

```dockerfile
FROM hf.co/unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF:UD-Q4_K_XL
PARAMETER num_ctx 32768
PARAMETER temperature 0.05
PARAMETER top_p 0.9
SYSTEM """
Tu es expert SQL. Règles strictes :
1. Tu utilises UNIQUEMENT les tables et colonnes fournies dans le schéma
2. Tu n'inventes JAMAIS de noms de colonnes
3. Tu précises toujours le dialecte SQL utilisé
4. Tu commentes les CTEs complexes
5. Tu renvoies uniquement le SQL, sans markdown ni explication
"""
```

```bash
ollama create sql-strict -f Modelfile
```

### dbt assistant

```dockerfile
FROM hf.co/unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF:UD-Q4_K_XL
PARAMETER num_ctx 32768
PARAMETER temperature 0.1
SYSTEM """
Tu es expert dbt. Conventions :
- Models staging : prefix stg_, materialized=view
- Models intermediate : prefix int_, materialized=ephemeral
- Models marts : prefix fct_/dim_, materialized=table
- Toujours utiliser ref() pour les dépendances
- Toujours utiliser source() pour les tables raw
- Inclure schema.yml avec tests : not_null, unique, relationships
- Inclure description pour chaque colonne
- Suivre les bonnes pratiques dbt 2026 (config, contrats, versions)
"""
```

---

## 🧪 Benchmarker sur ton schéma

Avant de t'engager sur un modèle, **toujours tester sur ton vrai schéma** avec un set de 10-20 questions business représentatives.

```python
test_set = [
    ("Top 5 produits par revenu Q1 2026", expected_sql_1),
    ("Taux de churn mensuel sur 12 mois", expected_sql_2),
    # ...
]

models = ["qwen3-coder:30b-data", "deepseek-coder-v2:16b-data", "qwen2.5-coder:14b-data"]
results = {}
for model in models:
    results[model] = []
    for question, expected in test_set:
        generated = ollama.generate(model=model, prompt=prompt_with(question))
        results[model].append(validate(generated, expected))

# Comparer les taux de succès
```

---

## Configuration OpenCode

```bash
# SQL / dbt
opencode --model ollama/qwen3-coder:30b-data

# Pandas / polars / Spark
opencode --model ollama/deepseek-coder-v2:16b-data

# Schéma / migration / planification
opencode --model ollama/glm-4.7:data
```

---

## Sources

- [Which LLM writes the best analytical SQL? — Tinybird](https://www.tinybird.co/blog/which-llm-writes-the-best-sql)
- [9 Best LLMs for SQL Generation 2026 — VisionVix](https://visionvix.com/best-llm-for-sql-generation/)
- [Text-to-SQL: Comparison of LLM Accuracy 2026 — AImultiple](https://research.aimultiple.com/text-to-sql/)
- [Best LLM for Data Analysis 2026 — Decodes Future](https://www.decodesfuture.com/articles/best-llm-for-data-analysis-2026-review)
- [Best LLM for Coding 2026 — WhatLLM](https://whatllm.org/best-llm-for-coding)
- [Best LLM for Coding 2026 — Onyx AI](https://onyx.app/best-llm-for-coding)
- [Best LLM for Data Analysis 2026 — ZenMux](https://zenmux.ai/blog/best-llm-for-data-analysis-in-2026-top-ai-models-for-accurate-insights)
