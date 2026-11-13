# ai-orchestration

Une orchestration d'agents pour **Claude Code** et **Codex** : un **Maestro** et **sept agents spécialisés**.

Vous parlez à une seule discussion, le Maestro. Il choisit l'agent et le modèle adaptés, délègue le travail, vérifie le résultat par une preuve et vous rend compte. Ce dépôt contient les consignes prêtes à installer, la documentation et un skill pour mettre à jour les modèles.

Dépôt : https://github.com/zaib-khan/ai-orchestration

## Le principe

- **Le Maestro** est la discussion principale. Il dialogue avec vous, choisit l'agent et le modèle, délègue, vérifie et rend compte. Il ne fait lui-même que les petites actions.
- **Sept agents**, un par **action** : coder, déployer, analyser, relire, chercher, rédiger, designer. On choisit l'agent selon l'action demandée, pas selon l'outil utilisé.
- **Un modèle par rôle**, choisi d'après des benchmarks publics : un modèle puissant là où il faut réfléchir, un modèle rapide et bon marché là où il faut chercher.
- **Vérifier par la preuve** : le Maestro ne dit jamais « c'est fait » sur la seule parole d'un agent. Il contrôle le diff et les tests, le fichier, le brouillon ou les sources.
- **Compte rendu** toujours dans le même ordre : le résultat, qui a travaillé (agent et modèle), la preuve, ce qui reste, les questions (une à la fois).

## Les agents

| Agent | Action | Claude Code (modèle, effort) | Codex (modèle, effort) |
|---|---|---|---|
| Maestro | dialoguer, choisir, déléguer, vérifier, rendre compte | Opus 5.5, moyen | GPT-6.1 Sol, medium |
| Codeur | écrire et corriger du code | Opus 5.5, moyen | GPT-6.1 Terra, medium |
| Infra | déployer, infrastructure, serveurs, CI/CD | Sonnet 5.5, moyen ; Opus 5.5, élevé si complexe ou production | GPT-6.1 Sol, medium ; GPT-6 Astra, high si complexe ou production |
| Analyste | analyser et conclure (lecture seule) | Opus 5.5, élevé ; Fable seulement avec accord | GPT-6.1 Sol, high |
| Relecteur | relire un travail fini (sur demande) | Sonnet 5.5, moyen (texte) ; Opus 5.5, élevé (code, configuration) | GPT-6.1 Sol, medium (texte) ; GPT-6 Astra, high (code, configuration) |
| Chercheur | trouver une information (lecture seule) | Haiku 5.5, moyen (fichiers) ; Sonnet 5.5, moyen (web) | GPT-6.1 Terra, low (fichiers) ; GPT-6.1 Sol, medium (web) |
| Rédacteur | écrire des textes en brouillon | Sonnet 5.5, moyen ; Opus 5.5, élevé (textes importants) | GPT-6.1 Sol, medium ; GPT-6 Astra, high (textes importants) |
| Designer | logo, identité visuelle, maquettes | Opus 5.5, élevé | GPT-6.1 Sol, high |

Fiches détaillées : [docs/agents.md](docs/agents.md). Raisons des choix et chiffres : [docs/choix-des-modeles.md](docs/choix-des-modeles.md).

## Règles communes

- **Petites actions seulement** pour le Maestro : 1 à 3 étapes, sans expertise ni risque (lire un fichier, `git status`, corriger une ou deux lignes, créer un petit fichier, commiter). Côté Codex : au plus 3 commandes par tâche. Au-delà, il délègue.
- **Brief complet** : un sous-agent ne connaît que la consigne du Maestro. Le Maestro donne le projet, le but, les fichiers, les contraintes (règle Git, production, tests) et ce que « fini » veut dire.
- **Questions** : les agents ne vous parlent pas directement. Le Maestro vous pose leurs questions, une à la fois.
- **Relecteur seulement sur demande** : jamais lancé automatiquement. À la fin d'une tâche, le Maestro demande « Tu veux une relecture ? » seulement si la tâche :
  1. touche au moins 10 fichiers ;
  2. touche la production ;
  3. produit un texte important (avocat, client, document officiel) ;
  4. est une analyse qui servira à prendre une décision ;
  5. concerne un sujet sensible, même avec peu de fichiers : connexion / authentification, sécurité, base de données.

  Sinon, le Maestro vérifie lui-même par la preuve. Le Relecteur ne corrige jamais : ses remarques repartent vers l'agent auteur. Deux tours au maximum, puis vous décidez.
- **Modèle premium seulement avec votre accord** : jamais Fable côté Claude, jamais GPT-6 Astra en effort xhigh côté Codex, sans votre accord pour la tâche.
- **Tests** : le Codeur demande d'abord s'il faut écrire des tests et pour quelle partie.
- **Grosse tâche** : le Codeur s'arrête et propose un plan.
- **Infra** : accord avant toute action destructrice ou irréversible, et avant chaque commande en production.
- **Git** : chaque projet décrit sa règle Git dans son CLAUDE.md ou AGENTS.md. Sans règle, l'agent s'arrête et demande. Aucun agent ne commite ou ne pousse sans demande.
- **Pas de mention de l'IA dans les commits** ni dans les descriptions de merge request / pull request (aucune ligne « Co-Authored-By », aucun « Generated with… »).
- **Envoi seulement avec accord** : aucun mail ni message n'est envoyé sans votre accord pour ce message. Le Rédacteur ne fait que des brouillons.
- **Design → code** : le Designer remet ses maquettes au Maestro, qui décide d'envoyer ou non le Codeur.
- **Rien d'inventé** : un agent signale ce qui manque au lieu de deviner.

## Installation rapide

> **Attention** : les copies ci-dessous **remplacent** des fichiers existants (`~/.claude/MAESTRO.md`, les agents qui portent le même nom, `~/.codex/AGENTS.md`). Faites toujours la sauvegarde d'abord. Guide complet, avec personnalisation, test et désinstallation : [docs/installation.md](docs/installation.md).

Lancez ces commandes à la racine du dépôt cloné.

**Claude Code**

```bash
# 1. Sauvegarde, dans un dossier horodaté
BACKUP=~/.claude/backups/ai-orchestration-$(date +%Y%m%d-%H%M%S)
mkdir -p "$BACKUP"
cp -R ~/.claude/agents "$BACKUP"/ 2>/dev/null
cp ~/.claude/MAESTRO.md "$BACKUP"/ 2>/dev/null
cp ~/.claude/CLAUDE.md "$BACKUP"/ 2>/dev/null

# 2. Copie
mkdir -p ~/.claude/agents
cp claude/agents/*.md ~/.claude/agents/
cp claude/MAESTRO.md ~/.claude/MAESTRO.md

# 3. Chargement du Maestro (ajouté une seule fois)
grep -qx '@MAESTRO.md' ~/.claude/CLAUDE.md 2>/dev/null || printf '\n@MAESTRO.md\n' >> ~/.claude/CLAUDE.md
```

Ouvrez ensuite une nouvelle session Claude Code. Si le dossier `~/.claude/agents/` vient d'être créé, redémarrez complètement Claude Code.

**Codex**

```bash
# 1. Sauvegarde, dans un dossier horodaté
BACKUP=~/.codex/backups/ai-orchestration-$(date +%Y%m%d-%H%M%S)
mkdir -p "$BACKUP"
cp -R ~/.codex/agents "$BACKUP"/ 2>/dev/null
cp ~/.codex/AGENTS.md "$BACKUP"/ 2>/dev/null
cp ~/.codex/config.toml "$BACKUP"/ 2>/dev/null

# 2. Copie
mkdir -p ~/.codex/agents
cp codex/AGENTS.md ~/.codex/AGENTS.md
cp codex/agents/*.toml ~/.codex/agents/
```

Puis, dans `~/.codex/config.toml`, réglez ces deux clés. Ce sont des clés de **premier niveau** : placez-les **avant** toute section `[...]` du fichier.

```toml
model = "gpt-6.1-sol"
model_reasoning_effort = "medium"
```

Vérifiez les modèles disponibles avec `/model` dans Codex.

## Le skill `update-agent-models`

Les benchmarks changent vite. Ce skill relit LiveBench et Artificial Analysis, puis propose, agent par agent, de garder ou de changer le modèle, côté Claude et côté Codex :

1. un sous-agent relève les chiffres (en lecture seule) ;
2. le skill revérifie les chiffres douteux, puis vous propose un tableau « modèle actuel / modèle suggéré / pourquoi » ;
3. vous validez chaque changement, un par un ;
4. il met à jour la documentation choisie (ce dépôt, fichiers installables compris, ou votre propre page), puis, seulement avec votre accord, les fichiers installés.

Où le copier :

| Outil | Pour tous vos projets | Pour un seul projet |
|---|---|---|
| Claude Code | `~/.claude/skills/update-agent-models/SKILL.md` | `.claude/skills/update-agent-models/SKILL.md` |
| Codex | vérifiez l'emplacement global dans la documentation de votre version de Codex | `.agents/skills/update-agent-models/SKILL.md` |

Exemple pour Claude Code, pour tous vos projets :

```bash
mkdir -p ~/.claude/skills/update-agent-models
cp skills/update-agent-models/SKILL.md ~/.claude/skills/update-agent-models/
```

Lancement : `/update-agent-models` (Claude Code) ou `$update-agent-models` (Codex), ou simplement « mets à jour les modèles des agents ».

Le skill : [skills/update-agent-models/SKILL.md](skills/update-agent-models/SKILL.md).

## Arborescence

```text
ai-orchestration/
├── README.md                     ce fichier
├── LICENSE                       licence MIT
├── docs/
│   ├── agents.md                 les 8 fiches (Maestro + 7 agents)
│   ├── choix-des-modeles.md      critères, chiffres, raisons, évolutions
│   └── installation.md           installation pas à pas, personnalisation, test
├── claude/
│   ├── MAESTRO.md                consignes du Maestro (Claude Code)
│   └── agents/                   7 agents Claude Code (.md)
├── codex/
│   ├── AGENTS.md                 consignes du Maestro (Codex)
│   └── agents/                   7 agents Codex (.toml)
└── skills/
    └── update-agent-models/
        └── SKILL.md              skill de mise à jour des modèles
```

Fichiers à installer : [claude/MAESTRO.md](claude/MAESTRO.md), [claude/agents/](claude/agents/), [codex/AGENTS.md](codex/AGENTS.md), [codex/agents/](codex/agents/).

Les consignes des agents sont en anglais ; la documentation est en français. Les agents parlent et rendent compte en français (modifiable, voir [docs/installation.md](docs/installation.md#personnaliser)).

## Limites

- **Benchmarks datés** : les chiffres et les choix de modèles datent du **12 novembre 2026**. Relancez le skill `update-agent-models` de temps en temps.
- **Noms de modèles** : côté Codex, vérifiez les modèles réellement proposés avec `/model`. Un nom absent ou changé doit être corrigé dans `AGENTS.md` et les fichiers TOML.
- **Lecture seule technique non garantie partout** : côté Claude, l'Analyste, le Relecteur et le Chercheur n'ont pas les outils d'écriture, mais ils ont `Bash` ; côté Codex, l'effet réel de `sandbox_mode = "read-only"` n'a pas été vérifié dans toutes les versions, et le Relecteur n'a pas ce réglage. La lecture seule repose donc aussi sur les consignes.
- **Un modèle par défaut par fichier d'agent** : pour les rôles à plusieurs routes, c'est le Maestro qui passe le modèle et l'effort au lancement.
- **Rôles personnalisés dans Codex** : selon la version, le Maestro doit parfois lire le TOML du rôle et transmettre ses instructions lui-même.
- **Benchmarks génériques** : une seule exécution par suite ; ils ne remplacent pas un essai sur vos vraies tâches.

## Licence

MIT. Voir [LICENSE](LICENSE).
