---
name: update-agent-models
description: Check the LiveBench and Artificial Analysis benchmarks for Claude and Codex (OpenAI) models and propose, agent by agent, a model update for the Maestro orchestration (Maestro + 7 agents). Use when the user types /update-agent-models, $update-agent-models, or asks to update the agents' models from the benchmarks.
---

# update-agent-models

Keep the models of the Maestro orchestration agents aligned with the latest benchmarks, for Claude and for Codex. Invoking this skill is the user's permission to read the two benchmark sites (and only them). Talk to the user in their usual language.

## References

Use the reference the user keeps up to date. Ask once if it is not clear.

- Default: the ai-orchestration repository.
  - Current choices: the agents table of `README.md` (Claude and Codex columns) and the cards of `docs/agents.md`.
  - Justifications and previous figures: `docs/choix-des-modeles.md` (figures, model per agent, "Évolutions" section).
- Or: the user's own page or notes, if they keep their choices elsewhere (for example a wiki page). Read it and use it the same way.
- Installed files, read only (to check what really runs): `~/.claude/MAESTRO.md` and `~/.claude/agents/*.md` (Claude Code); `~/.codex/AGENTS.md` and `~/.codex/agents/*.toml` (Codex). Never edit them from this skill without the user's explicit agreement (see step 3).

## 1. Research (sub-agent)

Launch one sub-agent (Claude Code: the `chercheur` agent with model Sonnet and medium effort if installed, otherwise any sub-agent with Sonnet; Codex: a sub-agent if available, otherwise do it yourself). The sub-agent only reads and reports; it writes nothing. Give it:

- The two sites:
  - LiveBench: `https://livebench.ai/#/?org=Anthropic` and `https://livebench.ai/#/?org=OpenAI` (JavaScript page: use a browser tool and read the leaderboard text; note the release date).
  - Artificial Analysis: `https://artificialanalysis.ai/models` (figures are in the page's embedded data: model objects with `slug`, `name`, `intelligenceIndex`, `terminalBench40`, `frontierCode20`, `itBenchSre`, `omniscienceHallucinationRate`, `price1mInputTokens`, `price1mOutputTokens`, `outputSpeed`, `code2videoElo`, `expressiveness`, `organization`, `transparency`). Fields are often stored far from the model's name, and blocks of other models sit in between: for each figure, take the occurrence closest to that model's own `"name"` (shortest distance before it) and check it is not shared with another model. The personality fields (`expressiveness`, `organization`, `transparency`) and `outputSpeed` are the most error-prone: if two models get identical values, or a value cannot be tied to its model with certainty, mark it "douteux" and do not base a recommendation on it.
- The current choices (copy the Claude and Codex agents tables from the reference).
- What each role needs (criteria used so far):

| Agent | Main criteria |
|---|---|
| Maestro | agentic (LiveBench), reasoning, cost per task, speed |
| Codeur | coding (LiveBench), FrontierCode, cost |
| Infra | Terminal-Bench, ITBench SRE (complex cases), cost |
| Analyste | data analysis, reasoning, research |
| Relecteur | transparency, low hallucination (text); coding, and a model different from the Codeur's (code) |
| Chercheur | low hallucination, cost, speed (files); browser use, research (web) |
| Rédacteur | expressiveness, organization, transparency |
| Designer | design (LiveBench), Code2Video, 3D |

- What to return: the release/update date of each site; a table of scores per current model of each provider (Claude, OpenAI); new models and models no longer listed; for each agent and each provider: current model, suggested model (with effort), "garder" or "changer", and the reason with the figures. Report "non trouvé" instead of guessing. No personal data in any query.

## 2. Proposal to the user

Check the sub-agent's report against the reference (current models really are the ones written there, and the ones in the installed files if they exist; flag any difference). Before proposing any change, re-check on the site yourself the figures it relies on; drop a suggestion based on figures marked "douteux" or that do not match. If nothing should change, say so with the date of the benchmarks, then go to step 4 only if the user wants the new figures kept.

Otherwise show one table per provider, only the agents concerned:

| Agent | Modèle actuel | Modèle suggéré | Pourquoi |
|---|---|---|---|

Then ask for validation one change at a time, one short question each. Keep the access rules: a premium model (Fable, GPT-6 Astra in xhigh, or any new top-tier model) is used only with the user's agreement; propose extending that rule to any new premium model.

## 3. Nothing changes without validation

Refused changes are left as they are. Never touch installed agent files (`~/.claude/MAESTRO.md`, `~/.claude/agents/`, `~/.codex/AGENTS.md`, `~/.codex/agents/`, `~/.codex/config.toml`) during steps 1 to 3.

## 4. Apply (main agent only)

1. Update the chosen documentation:
   - ai-orchestration repository:
     - documentation: in `docs/choix-des-modeles.md`, the figures, the benchmark date, the model per agent and a dated line in the "Évolutions" section (`YYYY-MM-DD — A → B, reason`); in `README.md` and `docs/agents.md`, the model and effort of each changed agent;
     - installable files of the repository, so they stay consistent with the documentation: `claude/MAESTRO.md` (agents table), `claude/agents/*.md` (`model` / `effort` in the frontmatter), `codex/AGENTS.md` (routing table), `codex/agents/*.toml` (`model`, `model_reasoning_effort`). For the Codex Maestro, also update the recommended `model` / `model_reasoning_effort` values in `README.md` and `docs/installation.md`.
   - User's own page: the same information (tables, model line of each changed agent, dated evolution line, sources, update date), following that page's own rules.
2. If the documentation lives in a Git repository, follow its Git rule; commit only if the user asks, and never mention AI in the commit message.
3. Installed files: list exactly what would change (Claude: the table of `~/.claude/MAESTRO.md` and the `model` / `effort` lines of each agent file; Codex: the table of `~/.codex/AGENTS.md`, the `model` / `model_reasoning_effort` lines of each TOML, and, for the Maestro, `model` in `~/.codex/config.toml`, plus `model_reasoning_effort` in the same file if the Maestro's effort changes; both are top-level keys, before any `[...]` section). Ask the user for explicit agreement, back up the files, then apply only what was agreed. Remind the user to check the Codex model names with `/model` and to open a new Claude Code or Codex session (restart if needed).
4. Report: what changed, for which agent, current → new model, what was kept, and the files changed.
