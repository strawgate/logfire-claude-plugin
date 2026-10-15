# Pydantic Logfire for Claude Code

Instrument applications and infrastructure, investigate telemetry, and manage
[Pydantic Logfire](https://logfire.pydantic.dev/) from Claude Code.

## Features

- `/instrument` - detect frameworks and add Logfire instrumentation
- `logfire-setup` skill - authenticate, inspect the repository, and route to the right setup workflow
- `logfire-infrastructure` skill - monitor hosts, containers, Kubernetes, databases, and cloud infrastructure
- `logfire-evals` skill - add dataset-based evaluations for AI and agent behavior
- `/debug` - investigate errors using Logfire traces via MCP
- `/query` - query traces, logs, and metrics interactively or add query capabilities to code
- `logfire-ui` skill - build links to Logfire projects, traces, and Explore filters
- SKILL.md with core Logfire patterns (configure, instrument, structured logging, AI/LLM instrumentation)
- MCP server for querying Logfire data

## Query vs UI Routing

The Logfire plugin uses one primary workflow per request:

| User intent | Route |
| --- | --- |
| Analyze, query, count, summarize, compare, or find root cause | Use `logfire-query` and return analysis in chat. |
| Open, show in browser, live view, Explore, Logfire UI, or give a link | Use `logfire-ui` and open or return a Logfire URL without querying first. |
| Ambiguous prompts like "show recent errors" or "view logs" | Ask whether the user wants a UI view or query analysis. |
| Explicit combined prompts like "analyze this and give me a link" | Do the requested query work, then provide the relevant Logfire link. |

For UI requests that omit the organization/project, the agent should resolve the current project through Logfire MCP auth/current-project metadata before asking the user. Environment values such as `LOGFIRE_BASE_URL`, `LOGFIRE_URL`, and exporter config identify the platform/API base only; they are not enough to infer the target organization/project.

## Install In Claude Code

```bash
claude plugin marketplace add pydantic/skills
claude plugin install logfire@pydantic-skills
```

The Claude plugin provides `/instrument`, `/debug`, `/query`, and the Logfire MCP server.

## MCP

The Logfire MCP server is configured automatically when you install the plugin. By default it
targets the US region endpoint (`https://logfire-us.pydantic.dev/mcp`). The plugin's skills and
commands are region-independent; only the MCP server entry differs per region.

### EU region and self-hosted instances

Claude Code reads the endpoint from the `LOGFIRE_MCP_URL` environment variable, so set it in
the shell where you launch Claude Code:

```bash
export LOGFIRE_MCP_URL=https://logfire-eu.pydantic.dev/mcp
```

This needs plugin version 0.1.4 or later; on earlier versions the URL is hardcoded. Run
`claude plugin list` to see the installed version and the marketplace it came from, then update
and restart Claude Code:

```bash
claude plugin marketplace update pydantic-skills
claude plugin update logfire@pydantic-skills
```

If you installed from Anthropic's `claude-plugins-official` marketplace instead, substitute that
name. That marketplace pins a specific commit of this repository, so a release can take longer to
appear there than on `pydantic-skills`, which tracks `main`.

In Claude Code, `.mcp.json` configures the `logfire` MCP server.

The Logfire MCP server requires normal Logfire authentication, such as `logfire auth` or a suitable `LOGFIRE_TOKEN`.

## Source of Truth

This repository publishes the Claude Code distribution of the shared Logfire
plugin maintained in [`pydantic/skills`](https://github.com/pydantic/skills/tree/main/plugins/logfire).
Keep commands, skills, the manifest, and MCP configuration synchronized with
that directory. Codex and Cursor metadata remain in the shared repository.
