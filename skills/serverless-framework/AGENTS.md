# serverless-framework Knowledge

Generated: 2026-06-23

## OVERVIEW

`serverless-framework` is a domain reference skill for AWS Serverless Framework projects, centered on `serverless.yml`, CLI lifecycle, event sources, variables, and MCP-assisted debugging.

## STRUCTURE

```text
serverless-framework/
|-- SKILL.md
`-- references/
    |-- serverless-yml.md
    |-- cli.md
    |-- events.md
    |-- variables.md
    `-- mcp-tools.md
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Change trigger scope or top-level examples | `SKILL.md` | Keep AWS Lambda/API Gateway/Serverless Framework wording trigger-rich. |
| Edit YAML configuration guidance | `references/serverless-yml.md` | Main IaC contract. |
| Edit deploy/package/log commands | `references/cli.md` | Serverless CLI lifecycle. |
| Edit event source examples | `references/events.md` | HTTP, schedule, S3, SQS, SNS, streams, and related triggers. |
| Edit variable resolution rules | `references/variables.md` | Stage, env, file, SSM, Secrets Manager, and interpolation guidance. |
| Edit AI-assisted debugging guidance | `references/mcp-tools.md` | Serverless MCP Server material. |

## CONVENTIONS

- Keep examples focused on AWS and Serverless Framework unless the skill description is deliberately expanded.
- Treat `serverless.yml` as the central artifact; reference files should support that contract, not replace it with unrelated IaC tools.
- Keep CLI guidance factual and command-oriented. Do not imply this repo itself has deployable services.
- If runtime versions or AWS service behavior change, refresh the relevant reference file and the manifest description if the trigger scope changes.

## ANTI-PATTERNS

- Do not add repo-level build/test/deploy commands for the examples in this skill.
- Do not mix this skill with project-starter scaffolding defaults unless a reference explicitly cross-links them.
- Do not document secrets as committed config; `.env` examples are illustrative for downstream projects only.
