# Installation

Ce guide installe l'orchestration **globalement** : elle s'applique à tous vos projets. Vous pouvez installer la version Claude Code, la version Codex, ou les deux.

Avant de commencer :

- **Sauvegardez** toujours les fichiers que vous allez remplacer ou modifier (étape 1 de chaque partie). Chaque sauvegarde va dans un dossier horodaté : une nouvelle installation n'écrase jamais une sauvegarde précédente.
- Lancez les commandes à la racine de ce dépôt cloné.
- Fichiers concernés : [claude/MAESTRO.md](../claude/MAESTRO.md), [claude/agents/](../claude/agents/), [codex/AGENTS.md](../codex/AGENTS.md), [codex/agents/](../codex/agents/).

## Claude Code

### 1. Sauvegarder

```bash
BACKUP=~/.claude/backups/ai-orchestration-$(date +%Y%m%d-%H%M%S)
mkdir -p "$BACKUP"
cp -R ~/.claude/agents "$BACKUP"/ 2>/dev/null
cp ~/.claude/MAESTRO.md "$BACKUP"/ 2>/dev/null
cp ~/.claude/CLAUDE.md "$BACKUP"/ 2>/dev/null
echo "Sauvegarde : $BACKUP"
```

Notez le chemin affiché : il sert pour la désinstallation. Si vous avez déjà des agents portant le même nom (`codeur`, `infra`, `analyste`, `relecteur`, `chercheur`, `redacteur`, `designer`) ou un `~/.claude/MAESTRO.md`, ils seront remplacés.

### 2. Copier les agents

```bash
mkdir -p ~/.claude/agents
cp claude/agents/*.md ~/.claude/agents/
```

### 3. Copier les consignes du Maestro

```bash
cp claude/MAESTRO.md ~/.claude/MAESTRO.md
```

### 4. Charger le Maestro

La ligne suivante doit se trouver **une seule fois** dans `~/.claude/CLAUDE.md` (le fichier est créé s'il n'existe pas) :

```text
@MAESTRO.md
```

Cette commande vérifie d'abord avec `grep` que la ligne n'y est pas déjà, puis l'ajoute sur une nouvelle ligne :

```bash
grep -qx '@MAESTRO.md' ~/.claude/CLAUDE.md 2>/dev/null || printf '\n@MAESTRO.md\n' >> ~/.claude/CLAUDE.md
```

### 5. Ouvrir une nouvelle session

`~/.claude/CLAUDE.md` n'est lu qu'au démarrage d'une session : ouvrez dans tous les cas une nouvelle session Claude Code. Si le dossier `~/.claude/agents/` vient d'être créé, redémarrez complètement Claude Code pour qu'il le détecte. Ensuite, la commande `/agents` doit afficher les sept agents.

### Bon à savoir (Claude Code)

- Chaque fichier d'agent a **un seul** modèle et un seul effort par défaut (dans l'en-tête : `model`, `effort`). Pour les routes alternatives (Infra complexe, Relecteur code, Chercheur web, Rédacteur important), le Maestro passe le modèle et l'effort au lancement, comme le demande le tableau de [MAESTRO.md](../claude/MAESTRO.md).
- Le modèle du Maestro est celui de la discussion principale : choisissez Opus 5.5, effort moyen, avec `/model` ou le sélecteur de modèle de l'application (ou un autre modèle si vous préférez).
- L'Analyste, le Relecteur et le Chercheur n'ont pas les outils d'écriture (`Write`, `Edit`). Ils ont `Bash` pour les recherches et les tests : leur lecture seule repose donc aussi sur leur consigne.
- Le Codeur, l'Infra et le Designer n'ont pas de champ `tools` : ils ont tous les outils de la session.

## Codex

### 1. Sauvegarder

```bash
BACKUP=~/.codex/backups/ai-orchestration-$(date +%Y%m%d-%H%M%S)
mkdir -p "$BACKUP"
cp -R ~/.codex/agents "$BACKUP"/ 2>/dev/null
cp ~/.codex/AGENTS.md "$BACKUP"/ 2>/dev/null
cp ~/.codex/config.toml "$BACKUP"/ 2>/dev/null
echo "Sauvegarde : $BACKUP"
```

Notez le chemin affiché : il sert pour la désinstallation.

### 2. Copier les consignes du Maestro

```bash
mkdir -p ~/.codex/agents
cp codex/AGENTS.md ~/.codex/AGENTS.md
```

Attention : cette commande remplace votre `~/.codex/AGENTS.md`. Si vous y aviez déjà des consignes, fusionnez-les à la main (votre ancienne version est dans la sauvegarde).

### 3. Copier les agents

```bash
cp codex/agents/*.toml ~/.codex/agents/
```

### 4. Régler le modèle du Maestro

Dans `~/.codex/config.toml`, réglez ces deux lignes. Modifiez-les si elles existent déjà, sans toucher au reste du fichier :

```toml
model = "gpt-6.1-sol"
model_reasoning_effort = "medium"
```

Ce sont des clés de **premier niveau** : placez-les en haut du fichier, **avant** toute section `[...]` (par exemple `[profiles.…]` ou `[mcp_servers.…]`). Placées après une section, elles appartiendraient à cette section et ne seraient pas prises en compte comme réglage général.

### 5. Vérifier les modèles

Ouvrez une nouvelle session Codex et tapez `/model`. Vérifiez que `gpt-6.1-sol`, `gpt-6.1-terra` et `gpt-6-astra` sont proposés. Si un nom manque ou a changé, corrigez-le dans `~/.codex/AGENTS.md`, dans les fichiers `~/.codex/agents/*.toml` et dans `~/.codex/config.toml`.

### Bon à savoir (Codex)

- Selon votre version de Codex, l'outil qui lance les sous-agents ne sait pas toujours choisir un rôle personnalisé. Dans ce cas, les consignes demandent au Maestro de lire le fichier TOML du rôle et de transmettre ses instructions, avec le modèle et l'effort explicites.
- L'Analyste et le Chercheur ont `sandbox_mode = "read-only"`. L'effet réel de ce réglage n'a pas été vérifié dans toutes les versions de Codex. Le Relecteur n'a pas ce réglage, car ses tests peuvent avoir besoin d'écrire.
- La règle « au plus 3 commandes » pour le Maestro sert aussi à limiter la consommation de tokens.

## Règle Git par projet

Le Codeur ne devine jamais la règle Git : il lit le CLAUDE.md (Claude Code) ou l'AGENTS.md (Codex) du projet. Si rien n'est écrit, il s'arrête et demande. Ajoutez donc une courte règle dans chaque projet, par exemple :

```markdown
## Git

- Projet personnel : travailler directement sur `main` tant que je suis seul ; branches dès qu'il y a des collaborateurs.
```

ou :

```markdown
## Git

- Une carte ou un ticket = une branche = une merge request. Jamais de commit direct sur `main`.
```

## Personnaliser

- **Votre prénom** : les consignes parlent de « the user ». Vous pouvez le remplacer par votre prénom dans `MAESTRO.md`, `AGENTS.md` et les fichiers des agents.
- **La langue** : le Maestro parle en français et les agents rendent compte en français (« Talk to the user in French », « Report in French »). Changez ces phrases pour une autre langue. Certaines phrases françaises sont citées telles quelles (« Tu veux une relecture ? », « c'est fait », « pas trouvé », « bloquant, important, détail ») : traduisez-les aussi si besoin.
- **Wiki** : la règle 2 de [MAESTRO.md](../claude/MAESTRO.md) et une partie du Rédacteur et du Chercheur concernent un wiki personnel de type LLM Wiki (index, journal, pages de domaine, `user.md`). Si vous n'en avez pas, vous pouvez les laisser : elles ne s'appliquent que dans un tel wiki.
- **Rappels** : le Maestro peut « créer un rappel To Do ». Cela suppose un outil de rappels de votre côté ; retirez la mention si vous n'en avez pas.
- **Modèles** : changez le modèle et l'effort dans l'en-tête des agents Claude, dans les fichiers TOML et dans les tableaux du Maestro. Gardez les tableaux et les fichiers cohérents. Le skill `update-agent-models` peut vous aider (voir le [README](../README.md#le-skill-update-agent-models)).

## Test conseillé

Après l'installation, dans un projet de test :

1. Demandez une recherche simple, par exemple « Où est défini le point d'entrée de ce projet ? ». Le Maestro doit lancer le Chercheur et indiquer dans son compte rendu l'agent et le modèle utilisés, avec les sources.
2. Demandez une petite correction de code. Le Codeur doit d'abord demander, via le Maestro, s'il faut écrire des tests, et s'arrêter si aucune règle Git n'est écrite dans le projet.
3. Demandez une action de 1 à 3 étapes (par exemple « lis le README »). Le Maestro doit la faire lui-même, sans déléguer.
4. Vérifiez qu'aucun Relecteur n'est lancé sans votre accord.

## Désinstaller

Remplacez `<sauvegarde>` par le chemin affiché à l'étape 1. Vérifiez le contenu de la sauvegarde avant de supprimer quoi que ce soit.

### Claude Code

1. Supprimez les sept agents copiés :

   ```bash
   rm ~/.claude/agents/{codeur,infra,analyste,relecteur,chercheur,redacteur,designer}.md
   ```

2. Remettez vos anciens agents s'ils étaient dans la sauvegarde :

   ```bash
   cp <sauvegarde>/agents/*.md ~/.claude/agents/ 2>/dev/null
   ```

3. Remettez votre ancien `~/.claude/MAESTRO.md` s'il existait (`cp <sauvegarde>/MAESTRO.md ~/.claude/`), sinon supprimez-le (`rm ~/.claude/MAESTRO.md`).
4. Retirez la ligne `@MAESTRO.md` de `~/.claude/CLAUDE.md`, ou remettez l'ancien fichier (`cp <sauvegarde>/CLAUDE.md ~/.claude/`).
5. Ouvrez une nouvelle session Claude Code.

### Codex

1. Supprimez les sept agents copiés :

   ```bash
   rm ~/.codex/agents/{codeur,infra,analyste,relecteur,chercheur,redacteur,designer}.toml
   ```

2. Remettez vos anciens agents s'ils étaient dans la sauvegarde :

   ```bash
   cp <sauvegarde>/agents/*.toml ~/.codex/agents/ 2>/dev/null
   ```

3. Remettez votre ancien `~/.codex/AGENTS.md` s'il existait (`cp <sauvegarde>/AGENTS.md ~/.codex/`), sinon supprimez-le (`rm ~/.codex/AGENTS.md`).
4. Remettez votre ancien `~/.codex/config.toml` (`cp <sauvegarde>/config.toml ~/.codex/`), ou remettez à la main vos anciennes valeurs de `model` et `model_reasoning_effort`. Si vous avez modifié d'autres réglages depuis l'installation, préférez la correction à la main pour ne pas les perdre.
5. Ouvrez une nouvelle session Codex.
