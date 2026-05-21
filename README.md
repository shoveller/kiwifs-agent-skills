# KiwiFS Agent Skills

[![skills.sh](https://skills.sh/b/shoveller/kiwifs-agent-skills)](https://skills.sh/shoveller/kiwifs-agent-skills)

Portable Agent Skills for working with KiwiFS knowledge bases through MCP.

This repository follows the `skills.sh` / `npx skills` layout: each skill lives under `skills/<skill-name>/SKILL.md`. The skill content is intentionally generic and does not depend on a private vault, host, or server name.

## Installation

Install all skills from this package:

```bash
npx skills add shoveller/kiwifs-agent-skills
```

Install only the KiwiFS skill:

```bash
npx skills add shoveller/kiwifs-agent-skills --skill kiwifs
```

## Available Skills

| Skill | Description |
|---|---|
| [kiwifs](./skills/kiwifs/SKILL.md) | Operate a KiwiFS knowledge base through MCP: markdown CRUD, search, graph traversal, workflows/Kanban, analytics, imports, drafts, and quality checks. |

## Requirements

The consuming agent needs access to a KiwiFS MCP server or equivalent tools. Tool names vary by runtime and server name, but commonly include prefixes such as:

```text
kiwi_*
kiwifs_*
mcp_kiwifs_*
mcp_<server>_kiwi_*
```

## Usage

After installation, ask your agent to load the skill explicitly if needed:

```text
/kiwifs
```

Then ask it to search, read, write, maintain, or operate your KiwiFS-backed knowledge base.

## Public-content policy

This package is public and intentionally avoids private hostnames, vault paths, credentials, personal runbooks, or deployment-specific assumptions. Keep downstream customization in your own private agent configuration rather than in this repository.

## License

MIT
