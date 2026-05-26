# agent-skills

Useful skills for coding agents. This repository contains custom agent skills that can be installed with the [skills.sh](https://skills.sh) CLI.

## Skills

| Skill | Description |
|-------|-------------|
| [project-starter](skills/project-starter) | Scaffold new TypeScript projects with Vite, React, MUI, ESLint, Prettier, Vitest, and optional AI tooling |
| [langchain-ts](skills/langchain-ts) | Comprehensive guide for building AI agents with LangChain TypeScript SDK |
| [mui](skills/mui) | Material-UI v9 component library patterns, styling, and theme integration |
| [serverless-framework](skills/serverless-framework) | AWS Lambda, API Gateway, and serverless application development |

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

## Skill Discovery

The [skills.sh](https://skills.sh) CLI automatically discovers skills in the `skills/` directory. Each skill is a folder containing a `SKILL.md` file with YAML frontmatter defining its `name` and `description`.
