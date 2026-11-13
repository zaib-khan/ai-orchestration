# Choix des modèles

Ce document explique pourquoi chaque agent utilise tel modèle et tel effort, côté Claude Code et côté Codex. Les chiffres viennent de deux benchmarks publics, relevés le **12 novembre 2026** :

- **LiveBench 2.0** (livebench.ai, release du 2026-11-06), filtres Anthropic et OpenAI ;
- **Artificial Analysis** (artificialanalysis.ai/models, Intelligence Index v5.0.1, tests à l'effort maximal).

Ces chiffres vieillissent vite. Pour les remettre à jour, utilisez le skill `update-agent-models` (voir le [README](../README.md#le-skill-update-agent-models)).

## Critères par rôle

| Agent | Critères principaux |
|---|---|
| Maestro | agentique (LiveBench), raisonnement, coût par tâche, vitesse |
| Codeur | code (LiveBench), FrontierCode, coût |
| Infra | Terminal-Bench, ITBench SRE (cas complexes), coût |
| Analyste | analyse de données, raisonnement, recherche |
| Relecteur | transparence et peu d'hallucinations (texte) ; code, et un modèle différent de celui du Codeur (code) |
| Chercheur | peu d'hallucinations, coût, vitesse (fichiers) ; navigation web, recherche (web) |
| Rédacteur | expressivité, organisation, transparence |
| Designer | design (LiveBench), Code2Video, 3D |

Le critère 3D n'a pas été relevé lors de la mesure du 12 novembre 2026. Il est prévu pour les prochaines mises à jour.

## Claude (Claude Code)

### Chiffres LiveBench (filtre Anthropic)

| | Fable 5.2 | Opus 5.5 | Sonnet 5.5 | Haiku 5.5 |
|---|---|---|---|---|
| Global | **84,9** | 83,4 | 77,0 | 68,9 |
| Agentique | **82,9** | 72,8 | 63,4 | 47,9 |
| Code | 79,2 | **87,1** | 77,2 | 77,6 |
| Design | **85,1** | 83,3 | 81,3 | 79,3 |
| Navigation web | 86,9 | 88,4 | **89,3** | 69,5 |
| Analyse de données | **81,5** | 80,3 | 59,5 | 53,1 |
| Recherche | **94,5** | 88,9 | 88,0 | 84,3 |
| Raisonnement | 88,8 | **92,2** | 91,6 | 82,4 |
| Coût par tâche réussie | 9,85 $ | 6,72 $ | 5,80 $ | **1,26 $** |

Le score global LiveBench est une moyenne pondérée : agentique ×2, design ×1,5, code ×1,5, le reste ×1.

### Chiffres Artificial Analysis

| | Fable 5.2 | Opus 5.5 | Sonnet 5.5 | Haiku 5.5 |
|---|---|---|---|---|
| Intelligence Index | **65** | 59 | 55 | 36 |
| Terminal-Bench 4.0 | 59,6 % | 59,6 % | **63,6 %** | 32,8 % |
| FrontierCode 2.0 | **48,8 %** | 46,4 % | 41,7 % | 34,9 % |
| Exactitude des connaissances | **79 %** | 66 % | 51 % | 35 % |
| Taux d'hallucination (plus bas = mieux) | 59 % | 50 % | 43 % | **34 %** |
| Prix entrée / sortie ($ par million de tokens) | 8 / 40 | 4 / 20 | 2 / 10 | **0,1 / 0,5** |
| Vitesse (tokens/s) | 96 | 115 | 146 | **256** |
| Code2Video (Elo) | 1127 | **1536** | 1443 | 1192 |

Personnalité (percentiles AA-PersonalityBench) : Opus 5.5 expressivité 90, organisation 77, transparence 90 ; Sonnet 5.5 expressivité 73, organisation 81, transparence 100.

### Constats

- Opus bat Fable en code (87,1 contre 79,2) pour deux fois moins cher. Fable ne domine que l'agentique long et la recherche.
- Sonnet est le meilleur au terminal (Terminal-Bench) et en navigation web, mais faible en analyse de données.
- Haiku code presque comme Sonnet et reste correct en design. Il connaît peu de choses de mémoire et tient mal les longues tâches : il doit chercher dans les sources, pas répondre de mémoire.

### Modèle par agent

| Agent | Modèle et effort | Raison |
|---|---|---|
| Maestro | Opus 5.5, moyen | c'est la discussion principale : son modèle est celui choisi dans l'application. Fable domine l'agentique mais coûte plus cher |
| Codeur | Opus 5.5, moyen | meilleur score en code, coût raisonnable |
| Infra | Sonnet 5.5, moyen ; Opus 5.5, élevé si complexe ou production | meilleur au terminal, deux fois moins cher qu'Opus |
| Analyste | Opus 5.5, élevé ; Fable 5.2 seulement avec accord | Sonnet trop faible en analyse de données |
| Relecteur | Sonnet 5.5, moyen (texte) ; Opus 5.5, élevé (code, configuration) | Sonnet est transparent et hallucine peu, mais il est moins fort en code |
| Chercheur | Haiku 5.5, moyen (fichiers) ; Sonnet 5.5, moyen (web) | Haiku est rapide et bon marché, et hallucine le moins ; Sonnet est meilleur en navigation |
| Rédacteur | Sonnet 5.5, moyen ; Opus 5.5, élevé (textes importants) | écriture claire et organisée, coût moyen ; Opus a la meilleure expressivité |
| Designer | Opus 5.5, élevé | écarts faibles en design, mais Opus nettement devant pour le design fait en code (Code2Video) |

## OpenAI (Codex)

### Chiffres LiveBench (filtre OpenAI)

| | GPT-6 Astra | GPT-6.1 Sol | GPT-6.1 Terra | GPT-6 Luna |
|---|---|---|---|---|
| Global | **84,5** | 81,4 | 78,8 | 60,4 |
| Agentique | **75,6** | 65,6 | 56,2 | 24,7 |
| Code | 87,5 | 82,1 | **89,2** | 70,8 |
| Design | **80,0** | 79,6 | 79,7 | 76,1 |
| Navigation web | 91,3 | **93,3** | 93,1 | 70,3 |
| Analyse de données | **83,0** | 82,7 | 82,3 | 73,4 |
| Recherche | **90,6** | 90,0 | 79,1 | 48,5 |
| Raisonnement | **92,7** | 92,6 | 88,7 | 81,8 |
| Coût par tâche réussie | 5,32 $ | 0,82 $ | 0,67 $ | **0,13 $** |

GPT-5.6 Terra (ancienne génération : 67,2 au global, 2,09 $ par tâche réussie) est écarté.

### Chiffres Artificial Analysis

| | GPT-6 Astra | GPT-6.1 Sol | GPT-6.1 Terra | GPT-6 Luna |
|---|---|---|---|---|
| Intelligence Index | **57** | 55 | 51 | 28 |
| Terminal-Bench 4.0 | **59,1 %** | 56,1 % | 55,6 % | 12,6 % |
| FrontierCode 2.0 | **42,4 %** | 40,6 % | 42,3 % | 25,2 % |
| ITBench SRE | 48,6 % | non relevé | non relevé | non relevé |
| Exactitude des connaissances | **62 %** | **62 %** | 48 % | 40 % |
| Taux d'hallucination (plus bas = mieux) | 50 % | 52 % | **35 %** | 79 % |
| Prix entrée / sortie ($ par million de tokens) | 10 / 50 | 2 / 10 | 1 / 5 | **0,1 / 0,5** |
| Vitesse (tokens/s) | 35 | 51 | 70 | **108** |
| Code2Video (Elo) | **1427** | 1369 | 1021 | 1030 |
| Expressivité / organisation / transparence (percentiles) | 59 / 69 / **94** | 53 / **79** / 92 | 20 / 67 / 42 | 40 / 22 / 53 |

Aucun modèle « Codex » récent n'était listé sur Artificial Analysis à cette date (le dernier était GPT-5.3 Codex).

### Constats

- GPT-6.1 Terra a le meilleur score en code de tous les modèles relevés, Claude compris (89,2 contre 87,1 pour Opus 5.5), pour 8 fois moins cher qu'Astra.
- GPT-6.1 Sol est le meilleur rapport qualité-prix : presque Astra en recherche, analyse et raisonnement, pour 6 fois moins cher par tâche.
- GPT-6 Luna n'est pas l'équivalent de Haiku : 79 % d'hallucinations, 12,6 % au terminal, 48,5 en recherche. Il n'est utilisé pour aucun agent.
- Astra est lent (35 tokens/s) et le plus cher au token, mais moins cher qu'Opus 5.5 par tâche réussie, car il consomme peu de tokens.

### Modèle par agent (efforts Codex : low, medium, high, xhigh)

| Agent | Modèle et effort | Raison |
|---|---|---|
| Maestro | GPT-6.1 Sol, medium | bon niveau en raisonnement et en recherche, environ 5 fois moins cher au token qu'Astra (voir « Évolutions ») |
| Codeur | GPT-6.1 Terra, medium | meilleur en code, le moins cher |
| Infra | GPT-6.1 Sol, medium ; GPT-6 Astra, high si complexe ou production | même logique que Sonnet / Opus ; Astra est le meilleur en diagnostic de panne (ITBench SRE 48,6 %) |
| Analyste | GPT-6 Astra, high | meilleur en analyse de données et en raisonnement |
| Relecteur | GPT-6.1 Sol, medium (texte) ; GPT-6 Astra, high (code, configuration) | Sol est très transparent (92) ; pour le code, un modèle différent de celui du Codeur donne un vrai regard neuf |
| Chercheur | GPT-6.1 Terra, low (fichiers) ; GPT-6.1 Sol, medium (web) | Terra hallucine le moins ; Sol est le meilleur en navigation et presque le meilleur en recherche |
| Rédacteur | GPT-6.1 Sol, medium ; GPT-6 Astra, high (textes importants) | écriture claire et organisée |
| Designer | GPT-6 Astra, high | meilleur modèle OpenAI en design et en Code2Video |

### Claude ou Codex ?

- Claude reste meilleur pour le **Rédacteur** (expressivité : Sol 53, Sonnet 73, Opus 90), utile pour imiter le style de l'utilisateur.
- Claude reste meilleur pour le **Designer** (design 83,3 contre 80,0 ; Code2Video 1536 contre 1427).
- OpenAI n'a pas d'équivalent de Fable. La règle d'accord porte donc sur GPT-6 Astra en effort xhigh.

## Modèles premium : seulement avec accord

- Claude : jamais Fable sans l'accord explicite de l'utilisateur, pour la tâche concernée.
- Codex : jamais GPT-6 Astra en effort xhigh sans l'accord explicite de l'utilisateur. Dans cette version, la règle vise seulement l'effort xhigh. Vous pouvez l'étendre à d'autres efforts ou à tout nouveau modèle haut de gamme.

## Évolutions

- 2026-11-12 — Premier choix, avant les benchmarks : Infra Opus, Relecteur Sonnet, Chercheur Haiku. Après les benchmarks : Infra Sonnet (Opus si complexe), Relecteur Sonnet pour le texte et Opus pour le code, Chercheur Haiku pour les fichiers et Sonnet pour le web. Le Maestro est ajouté.
- 2026-11-12 — Variante Codex : Maestro sur GPT-6 Astra, medium (meilleur score agentique, 75,6).
- 2026-11-13 — Chercheur Claude : Haiku 5.5, effort bas → moyen, après un premier test réel.
- 2026-11-13 — Maestro Codex : GPT-6 Astra → GPT-6.1 Sol, effort medium, pour réduire la consommation de tokens. Astra coûte environ 5 fois le prix de Sol au token, et le Maestro, qui tient la discussion principale, en consomme une grande part. Les modèles et efforts par défaut sont fixés dans les fichiers TOML : Chercheur Terra low ; Infra, Rédacteur et Relecteur Sol medium. Les routes Astra (Infra complexe, Relecteur code, Analyste, Designer, Rédacteur important) restent inchangées. Le gain réel reste à mesurer.

## Limites des benchmarks

- Ce sont des tests génériques, avec une seule exécution par suite. Ils ne remplacent pas un essai sur vos vraies tâches.
- LiveBench est sponsorisé par Abacus.AI.
- Artificial Analysis teste à l'effort maximal. Un effort plus bas coûte moins cher mais réussit un peu moins.
- Sur Artificial Analysis, certains champs (personnalité, vitesse) sont difficiles à rattacher au bon modèle dans les données de la page. Le skill `update-agent-models` demande de les marquer « douteux » en cas de doute.
- Les noms de modèles et leur disponibilité changent. Côté Codex, vérifiez les modèles réellement proposés avec `/model`.
