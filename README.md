# agent-skills

Useful skills for coding agents. This repository contains custom agent skills that can be installed with the [skills.sh](https://skills.sh) CLI.

## Skills

| Skill | Description |
|-------|-------------|
| [project-starter](skills/project-starter) | Scaffold new TypeScript projects with Vite or Next.js, React, MUI, ESLint, Prettier, Vitest, AGENTS.md, backend logging, and optional AI tooling |
| [langchain-ts](skills/langchain-ts) | Comprehensive guide for building AI agents with LangChain TypeScript SDK |
| [mui](skills/mui) | Material-UI v9 component library patterns, styling, and theme integration |
| [serverless-framework](skills/serverless-framework) | AWS Lambda, API Gateway, and serverless application development |
| [grammy-bot-builder](skills/grammy-bot-builder) | Build Telegram bots with grammY (TypeScript/Node.js/Bun/Deno) |

## Install

Install all skills:

```bash
npx skills add <owner/repo> --all
```

Install a specific skill:

```bash
npx skills add <owner/repo> --skill langchain-ts
```

## Configuration

The `skills.sh.json` file in this repository serves as a manifest listing all available skills, their metadata, and file structures.

## OpenCode and OMO Configs

The `opencode-omo-config/` directory contains reusable OpenCode and OMO configuration files. These files are intended to be shared publicly: they contain model routing, agent definitions, fallback settings, prompt templates, and secret-file guardrails, but no private credentials or environment values.

| File | Purpose |
|------|---------|
| `opencode-omo-config/opencode.jsonc` | Example OpenCode config that enables the `oh-my-openagent@latest` plugin, defines Builder/Planner agent prompts, and denies access to `.env` files while allowing `.env.example` |
| `opencode-omo-config/omo.jsonc` | Default OMO multi-harness model routing and fallback config |
| `opencode-omo-config/omo-low-budget.jsonc` | OpenCode Go-first routing, with OpenAI reserved for the GPT-native Hephaestus agent |
| `opencode-omo-config/omo-mid-budget.jsonc` | The default routing strategy updated to prefer Kimi K3 for compatible orchestration and visual roles |
| `opencode-omo-config/omo-high-budget.jsonc` | OpenAI Pro and OpenCode Go routing with Venice models only at the end of compatible fallback chains and no Claude models |
| `opencode-omo-config/prompts/*.md` | Prompt templates referenced by the OpenCode agent definitions |

Copy `opencode.jsonc` into your OpenCode configuration location and copy one OMO profile to `~/.omo/omo.jsonc`, then adjust model/provider names for your authenticated subscriptions. The budget files are standalone alternatives, not files that OMO loads by those names. Run `opencode models` to confirm availability and `bunx oh-my-openagent doctor --verbose` after installation to inspect effective resolution.

The unified OMO schema permits a `[codex]` block, but Codex Light and OpenCode do not share model selection. Codex models and managed agent roles remain in `~/.codex/config.toml` and `~/.codex/agents/*.toml`; these OMO examples use `[codex]` only for the shared CodeGraph loader. Keep private API keys and local environment values outside this repository. See the current [agent/model matching](https://omo.dev/docs#agent-model-matching) and [configuration](https://omo.dev/docs#configuration) references before changing the routing.

## Skill Discovery

The [skills.sh](https://skills.sh) CLI automatically discovers skills in the `skills/` directory. Each skill is a folder containing a `SKILL.md` file with YAML frontmatter defining its `name` and `description`.
