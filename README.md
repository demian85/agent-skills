# agent-skills

Useful skills for coding agents. This repository contains custom agent skills that can be installed with the [skills.sh](https://skills.sh) CLI.

## Skills

| Skill | Description |
|-------|-------------|
| [project-starter](skills/project-starter) | Scaffold new TypeScript projects with Vite, React, MUI, ESLint, Prettier, Vitest, and optional AI tooling |
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

## OpenCode and oh-my-openagent Configs

The `opencode-omo-config/` directory contains reusable OpenCode and oh-my-openagent plugin configuration files. These files are intended to be shared publicly: they contain model routing, agent definitions, fallback settings, prompt templates, and secret-file guardrails, but no private credentials or environment values.

| File | Purpose |
|------|---------|
| `opencode-omo-config/opencode.jsonc` | Example OpenCode config that enables the `oh-my-openagent@latest` plugin, defines Builder/Planner agent prompts, and denies access to `.env` files while allowing `.env.example` |
| `opencode-omo-config/oh-my-openagent.jsonc` | Default oh-my-openagent model routing and fallback config |
| `opencode-omo-config/oh-my-openagent-low-budget.jsonc` | Lower-cost model routing profile |
| `opencode-omo-config/oh-my-openagent-mid-budget.jsonc` | Balanced model routing profile |
| `opencode-omo-config/oh-my-openagent-high-budget.jsonc` | Higher-capability model routing profile |
| `opencode-omo-config/prompts/*.md` | Prompt templates referenced by the OpenCode agent definitions |

Copy the files into your OpenCode configuration location and adjust model/provider names for your own setup. Keep private API keys and local environment values outside this repository.

## Skill Discovery

The [skills.sh](https://skills.sh) CLI automatically discovers skills in the `skills/` directory. Each skill is a folder containing a `SKILL.md` file with YAML frontmatter defining its `name` and `description`.
