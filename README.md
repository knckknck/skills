# Agent Skills

A collection of skills for AI coding agents. Skills are packaged instructions and scripts that extend agent capabilities.

Skills follow the [Agent Skills](https://agentskills.io/) format.

## Available Skills

### go-best-practices

Go best practices and style guide for Direkt (based on the Uber Go Style Guide).

**Use when:**
- Writing new Go programs or services
- Reviewing or refactoring Go code
- Fixing concurrency, correctness, or performance issues

**Categories covered:**
- Correctness & error handling
- Concurrency
- APIs & types
- Performance
- Readability & style
- Testing
- Tooling / linting

**Docs:**
- Skill: `skills/go-style-guide/SKILL.md`
- Rules (source): `skills/go-style-guide/rules/`
- Compiled guide: `skills/go-style-guide/AGENTS.md`

**References:**
- `skills/go-style-guide/metadata.json` (full reference list)
- Upstream: `https://github.com/uber-go/guide` and `https://go.uber.org/guide`

### brand-guidelines

Applies Direkt brand colors and typography to UI and non-code artifacts.

**Use when:**
- You need Direkt-consistent colors, typography, or visual styling
- You’re generating or reviewing docs/presentations/marketing assets

**Docs:**
- Skill: `skills/brand-guidelines/SKILL.md`
- License: `skills/brand-guidelines/LICENSE.txt`

### mcp-builder

Guide for building high-quality MCP (Model Context Protocol) servers and tool APIs.

**Use when:**
- Designing MCP tool surfaces (naming, schemas, pagination, error messages)
- Implementing MCP servers (especially in Go) and writing evaluations

**Docs:**
- Skill: `skills/mcp-builder/SKILL.md`
- Reference library: `skills/mcp-builder/reference/`

### numscript-guidelines

Numscript guidelines for modeling financial transactions with Numscript (DSL).

**Use when:**
- Writing or reviewing Numscript programs
- Modeling complex money movements and splits

**Docs:**
- Skill: `skills/numscript-guidelines/SKILL.md`
- Reference library: `skills/numscript-guidelines/reference/`

## Installation

```bash
npx add-skill direktly/agent-skills
```

## Usage

Skills are automatically available once installed. The agent will use them when relevant tasks are detected.

**Examples:**
```
Review this for performance issues
```

```
Help me optimize this API
```

## Skill Structure

Each skill contains:
- `SKILL.md` - Instructions for the agent
- `scripts/` - Helper scripts for automation (optional)
- `references/` - Supporting documentation (optional)
- `assets/` - Templates, resources, and other assets (optional)

## Creating a Skill

Skills are simple to create - just a folder with a SKILL.md file containing YAML frontmatter and instructions. You can use the template-skill in this repository as a starting point:

```md
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
---

# My Skill Name

[Add your instructions here that Claude will follow when this skill is active]

## Examples
- Example usage 1
- Example usage 2

## Guidelines
- Guideline 1
- Guideline 2
```

The frontmatter requires only two fields:

- `name` - A unique identifier for your skill (lowercase, hyphens for spaces)
- `description` - A complete description of what the skill does and when to use it

## License

[APACHE-2.0](LICENSE)
