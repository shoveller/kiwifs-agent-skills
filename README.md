# KiwiFS Agent Skills

[![skills.sh](https://skills.sh/b/shoveller/kiwifs-agent-skills)](https://skills.sh/shoveller/kiwifs-agent-skills)

Portable Agent Skills for working with [KiwiFS](https://www.kiwifs.com/) knowledge bases exposed through remote MCP servers.

KiwiFS is a git-backed knowledge-base and wiki system. This package is only the agent skill layer for operating an already-connected KiwiFS remote MCP server; for the KiwiFS product/project itself, see https://www.kiwifs.com/.

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

The consuming agent needs access to a remote KiwiFS MCP server, typically backed by a KiwiFS deployment such as the one described at https://www.kiwifs.com/, or to an agent runtime that exposes equivalent KiwiFS MCP tools. This package does not start, host, or configure KiwiFS itself; it only teaches the agent how to use already-connected remote MCP tools. Tool names vary by runtime and server name, but commonly include prefixes such as:

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

Then ask it to search, read, write, maintain, or operate your KiwiFS-backed knowledge base through the connected remote MCP server.

## Public-content policy

This package is public and intentionally avoids private hostnames, vault paths, credentials, personal runbooks, or deployment-specific assumptions. Keep downstream customization in your own private agent configuration rather than in this repository.

## License

MIT
