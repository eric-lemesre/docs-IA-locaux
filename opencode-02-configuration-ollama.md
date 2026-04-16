# OpenCode — Configuration avec Ollama (modèles locaux)

Configuration d'OpenCode pour utiliser **Ollama** comme provider local, avec **découverte automatique** des modèles via le plugin `opencode-models-discovery`.

> Prérequis : OpenCode installé (voir `opencode-01-installation.md`) et Ollama opérationnel.

---

## 1. Architecture de la configuration

OpenCode sépare **config** et **credentials** dans deux fichiers distincts. Les deux sont obligatoires, sinon le provider apparaît mais échoue avec une erreur d'auth.

| Fichier | Rôle |
|---|---|
| `~/.config/opencode/opencode.json` | Providers, modèles, plugins, comportement |
| `~/.local/share/opencode/auth.json` | Clés/tokens d'authentification |
| `~/.config/opencode/package.json` | Plugins npm/bun installés |

---

## 2. Installation du plugin de découverte automatique

Sans ce plugin, il faut lister manuellement chaque modèle Ollama dans `opencode.json`. Avec, ils sont détectés automatiquement via l'endpoint `GET /v1/models`.

```bash
cd ~/.config/opencode
bun add opencode-models-discovery
# ou : npm install opencode-models-discovery
```

Vérification :
```bash
cat ~/.config/opencode/package.json
```

Doit contenir :
```json
{
  "dependencies": {
    "opencode-models-discovery": "^0.6.1"
  }
}
```

---

## 3. Fichier de configuration principal

Crée `~/.config/opencode/opencode.json` :

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["opencode-models-discovery@latest"],
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://127.0.0.1:11434/v1"
      }
    }
  }
}
```

### Détail des champs

| Champ | Rôle |
|---|---|
| `$schema` | Active l'autocomplétion JSON dans les éditeurs |
| `plugin` | Liste des plugins à charger au démarrage |
| `provider.ollama` | Identifiant du provider (libre, sera utilisé partout) |
| `npm` | Adapter AI SDK : `@ai-sdk/openai-compatible` car Ollama expose une API compatible OpenAI |
| `name` | Libellé affiché dans l'UI |
| `options.baseURL` | Endpoint Ollama (`/v1` est obligatoire pour la compat OpenAI) |

> 💡 Sur **Windows**, remplacer `localhost`/`127.0.0.1` par `127.0.0.1` n'est pas suffisant : utiliser explicitement `http://127.0.0.1:11434/v1`.

---

## 4. Fichier d'authentification (placeholder)

Crée `~/.local/share/opencode/auth.json` :

```json
{
  "ollama": {
    "type": "api",
    "key": "ollama"
  }
}
```

⚠️ **La clé top-level (`ollama`) DOIT correspondre exactement au nom du provider dans `opencode.json`.** Sinon : provider visible mais erreur 401 au premier appel.

Sécuriser le fichier (recommandé même sans vraies clés) :
```bash
chmod 600 ~/.local/share/opencode/auth.json
```

---

## 5. Configuration avancée du plugin (optionnel)

Pour filtrer les providers scannés ou ajuster le cache :

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": [
    ["opencode-models-discovery", {
      "providers": {
        "include": ["ollama"],
        "exclude": []
      },
      "discovery": {
        "enabled": true,
        "ttl": 15000
      }
    }]
  ],
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://127.0.0.1:11434/v1"
      }
    }
  }
}
```

| Option | Rôle | Défaut |
|---|---|---|
| `providers.include` | Whitelist des providers à scanner | `[]` (tous) |
| `providers.exclude` | Blacklist (si `include` vide) | `[]` |
| `discovery.enabled` | Active la découverte | `true` |
| `discovery.ttl` | Cache en ms | `15000` |

---

## 6. ⚠️ Étape critique — Étendre le contexte des modèles

**Ollama force `num_ctx = 4096` par défaut**, ce qui casse les tool calls dès que le prompt système + l'historique dépasse cette taille. Pour de l'agentique, viser **16k minimum, 32k+ recommandé**.

### Créer une variante d'un modèle avec contexte étendu

```bash
ollama run qwen2.5-coder:7b
>>> /set parameter num_ctx 32768
>>> /save qwen2.5-coder:7b-32k
>>> /bye
```

Le nouveau modèle `qwen2.5-coder:7b-32k` apparaîtra automatiquement dans OpenCode au prochain démarrage (grâce au plugin de découverte).

### Recommandations par modèle (16 GB VRAM)

| Modèle source | Variante recommandée | `num_ctx` |
|---|---|---|
| `qwen2.5-coder:7b` | `qwen2.5-coder:7b-32k` | 32 768 |
| `devstral-small-2:latest` | `devstral-small-2:24k` | 24 576 |
| `gpt-oss:latest` | `gpt-oss:60k` | 65 536 |
| `qwen3-coder:30b-a3b` | `qwen3-coder:30b-a3b-32k` | 32 768 |

---

## 7. Lancement et sélection du modèle

### Mode interactif
```bash
opencode
```

Dans l'UI :
- Tape `/models` → la liste affiche **tous tes modèles Ollama** automatiquement
- Sélectionne celui à utiliser

### CLI direct (one-shot)
```bash
opencode --model ollama/qwen2.5-coder:7b-32k "génère un endpoint REST en Rust avec axum"
```

Format obligatoire : `<provider_id>/<model_id>`.

### Variable d'environnement (modèle par défaut)
```bash
export OPENCODE_MODEL="ollama/devstral-small-2:24k"
opencode
```

---

## 8. Vérification

### Tester l'endpoint Ollama
```bash
curl -s http://127.0.0.1:11434/v1/models | jq '.data[].id'
```

Tous les modèles listés ici seront découverts par OpenCode.

### Tester OpenCode
```bash
opencode --model ollama/qwen2.5-coder:7b "écris une fonction fibonacci en Python"
```

Si la réponse arrive → tout fonctionne.

---

## 9. Alternative : `ollama launch opencode`

Pour un setup **temporaire/automatique** sans modifier les fichiers de config :

```bash
ollama launch opencode
```

Ollama injecte sa configuration via la variable `OPENCODE_CONFIG_CONTENT` au démarrage. Pratique pour tester, mais :
- Les modèles définis dans `opencode.json` n'apparaissent pas dans le menu de sélection initial
- À chaque relance, il faut repasser par cette commande

Pour un usage quotidien, **garder la config statique avec le plugin** (sections 2-7).

---

## 10. Multi-providers (Ollama + autres)

Tu peux combiner Ollama avec d'autres providers locaux ou cloud :

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["opencode-models-discovery@latest"],
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": { "baseURL": "http://127.0.0.1:11434/v1" }
    },
    "lmstudio": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "LM Studio (local)",
      "options": { "baseURL": "http://127.0.0.1:1234/v1" }
    },
    "anthropic": {
      "npm": "@ai-sdk/anthropic"
    }
  }
}
```

Pour les providers cloud, ajouter la vraie clé dans `auth.json` :
```json
{
  "ollama":    { "type": "api", "key": "ollama" },
  "lmstudio":  { "type": "api", "key": "lmstudio" },
  "anthropic": { "type": "api", "key": "sk-ant-xxxxxxx" }
}
```

---

## 11. Dépannage configuration

| Symptôme | Cause | Solution |
|---|---|---|
| Provider absent de `/models` | OpenCode pas redémarré | Relancer après modif `opencode.json` |
| Erreur 401 / auth | Noms divergents entre les 2 fichiers | Vérifier que la clé top-level dans `auth.json` == nom dans `provider:` |
| `Cannot find module '@ai-sdk/openai-compatible'` | Plugin pas installé | `cd ~/.config/opencode && bun add @ai-sdk/openai-compatible` |
| Tool calls qui échouent / boucle infinie | `num_ctx` trop petit (4096) | Créer variante avec `num_ctx ≥ 16384` (section 6) |
| Modèles découverts mais réponses vides | `/v1/models` répond mais Ollama plante | `journalctl -u ollama -f` pour les logs |
| `Connection refused` | Ollama pas lancé | `sudo systemctl start ollama` |
| Sur Windows : `Unable to connect` | `localhost` mal résolu | Utiliser `http://127.0.0.1:11434/v1` |
| Modèle introuvable malgré `ollama list` | Cache plugin | `discovery.ttl: 0` ou redémarrer OpenCode |

---

## 12. Récap structure finale

```
~/.config/opencode/
├── opencode.json           ← config principale (providers, plugins)
├── package.json            ← dépendances npm/bun (plugins)
├── bun.lock                ← lockfile
└── node_modules/           ← plugins installés
    └── opencode-models-discovery/

~/.local/share/opencode/
├── auth.json               ← credentials (perms 600)
├── storage/                ← données persistantes
└── log/                    ← logs

~/.local/bin/
└── opencode -> /usr/bin/opencode-cli   (ou autre chemin selon install)
```

---

## Liens utiles

- [Documentation OpenCode — Providers](https://opencode.ai/docs/providers/)
- [Documentation OpenCode — Models](https://opencode.ai/docs/models/)
- [Plugin opencode-models-discovery](https://github.com/rivy-t/opencode-models-discovery)
- [Documentation Ollama — Intégration OpenCode](https://docs.ollama.com/integrations/opencode)
- [Issue #6231 — Auto-discover models (suivi de l'intégration native future)](https://github.com/anomalyco/opencode/issues/6231)
