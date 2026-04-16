# OpenCode — Installation (Linux)

Guide d'installation d'OpenCode et de ses dépendances pour l'utiliser avec des modèles locaux Ollama.

> Cible : Linux (Debian/Ubuntu et dérivés). Vérifié sur Debian 14 / Linux 6.19.

---

## 1. Prérequis

| Outil | Version min | Vérification |
|---|---|---|
| `curl` | n'importe | `curl --version` |
| `bun` ou `npm` | bun ≥ 1.3 / npm ≥ 9 | `bun --version` ou `npm --version` |
| Ollama | ≥ 0.18 | `ollama --version` |
| GPU NVIDIA | optionnel mais recommandé | `nvidia-smi` |

### Installer les prérequis manquants

**Bun** (recommandé, plus rapide que npm pour les plugins OpenCode) :
```bash
curl -fsSL https://bun.sh/install | bash
```

**Ollama** :
```bash
curl -fsSL https://ollama.com/install.sh | sh
sudo systemctl enable --now ollama
```

Vérifier que l'API Ollama répond :
```bash
curl http://localhost:11434/v1/models
```

---

## 2. Installation d'OpenCode

### Méthode A — Script officiel (recommandée)

```bash
curl -fsSL https://opencode.ai/install | bash
```

Le script installe le binaire natif (~150 MB, ELF x86-64) sous l'un de ces chemins selon ta distrib :

- `/usr/bin/opencode-cli` (paquet système)
- `~/.opencode/bin/opencode` (install user)
- `/usr/local/bin/opencode`

### Méthode B — Via npm/bun (alternative)

```bash
npm install -g opencode-ai
# ou
bun add -g opencode-ai
```

⚠️ Si tu utilises **nvm** : l'install est liée à la version de Node active au moment de l'installation. Si tu changes de version Node ensuite, le binaire ne sera plus dans le PATH.

---

## 3. Localiser le binaire après installation

```bash
# Recherche large
find / -maxdepth 6 -name "opencode*" -type f -executable 2>/dev/null

# Vérifier la version
which opencode || which opencode-cli
opencode --version  # ou opencode-cli --version
```

---

## 4. Cas particulier : binaire nommé `opencode-cli`

Sur certaines distributions, le paquet système installe `opencode-cli` au lieu de `opencode`. Pour avoir la commande `opencode` standard :

### Option 1 — Symlink user (sans sudo)
```bash
mkdir -p ~/.local/bin
ln -s /usr/bin/opencode-cli ~/.local/bin/opencode
```

Vérifier que `~/.local/bin` est dans le PATH (sinon ajouter à `~/.bashrc`) :
```bash
echo $PATH | tr ':' '\n' | grep -q "$HOME/.local/bin" || \
  echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Option 2 — Symlink global (avec sudo)
```bash
sudo ln -s /usr/bin/opencode-cli /usr/local/bin/opencode
```

### Option 3 — Alias shell
```bash
echo 'alias opencode="opencode-cli"' >> ~/.bashrc
source ~/.bashrc
```

---

## 5. Désinstaller les installations en double

Si tu as plusieurs versions installées (système + npm sous différents Node), supprime celles que tu n'utilises pas :

### npm sous une version spécifique de nvm
```bash
# Lister les installations npm globales pour une version Node donnée
PATH=~/.nvm/versions/node/v22.20.0/bin:$PATH npm ls -g --depth=0

# Désinstaller
PATH=~/.nvm/versions/node/v22.20.0/bin:$PATH npm uninstall -g opencode-ai
```

### Binaire système
```bash
sudo rm /usr/bin/opencode-cli
# ou via le gestionnaire de paquets si installé via .deb/.rpm
```

---

## 6. Création des dossiers de config

OpenCode utilise quatre dossiers (créés au premier lancement, mais on peut les préparer) :

```bash
mkdir -p ~/.config/opencode          # config (opencode.json, plugins)
mkdir -p ~/.local/share/opencode     # auth, storage, logs
mkdir -p ~/.local/state/opencode     # état
mkdir -p ~/.cache/opencode           # cache
```

---

## 7. Vérification finale

```bash
# Binaire accessible
which opencode && opencode --version

# Ollama opérationnel
ollama list
curl -s http://localhost:11434/v1/models | head -c 200

# Dossiers OpenCode présents
ls -d ~/.config/opencode ~/.local/share/opencode
```

Si toutes les commandes répondent → **OpenCode est installé**, passe au document de configuration.

---

## 8. Mise à jour ultérieure

```bash
# Si installé via script officiel
curl -fsSL https://opencode.ai/install | bash

# Si installé via npm/bun
npm update -g opencode-ai
bun update -g opencode-ai

# Si paquet système .deb / .rpm
sudo apt update && sudo apt upgrade opencode-cli
```

---

## Dépannage installation

| Problème | Solution |
|---|---|
| `command not found: opencode` | Vérifier le PATH, utiliser le symlink (étape 4) |
| `opencode: cannot execute binary file` | Mauvaise architecture — réinstaller pour la bonne (x86-64 / arm64) |
| Conflit npm avec nvm | Désinstaller toutes les versions, réinstaller via script officiel |
| `Permission denied` au lancement | `chmod +x` sur le binaire ou réinstaller proprement |
| Erreur `GLIBC` trop ancienne | Distribution trop ancienne — utiliser la version Node/npm |
