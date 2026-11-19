---
description: Lists the command-line tools (CLIs) installed in this environment for agent use, and when to use each one instead of its MCP equivalent. Use before calling an MCP server for a service that also has a CLI here, such as Vercel.
---

# External Tools

These CLIs are installed and authenticated in my environment. When a task needs one of these services and you already know which operation you need, use the CLI rather than the matching MCP server. The CLI call is direct and cheap, and you don't have to load tool schemas or look up IDs first.

Use the MCP server only when you don't yet know which operation you need, or when the CLI can't do the job.

Run CLIs through the Bash tool. Use non-interactive flags (such as `--yes`) and redirect stdin from `/dev/null`, so a prompt can't hang the session.

## vercel

The Vercel CLI. Use it for deployment status, project management, and environment variable configuration.

| Task | Command |
|---|---|
| Deployment status | `vercel ls` |
| Environment variables | `vercel env ls` |

Use this instead of the Vercel MCP in any session where you already know what operation you need.

Notes:

- `vercel ls` and `vercel env ls` use the project linked in the current directory (`.vercel/project.json`). If the directory isn't linked, pass the team with `--scope <team-slug>` and run the command from a directory that isn't linked to a different project. Otherwise the CLI may ask to link the directory.
- `vercel ls` doesn't support `--limit`.
- `vercel env ls` lists variable names only. Never print or decrypt variable values unless I ask for them.
- Check which account the CLI is using with `vercel whoami`.

<!-- Add more CLIs below as they are installed, one `## <cli-name>` section each, using the same format. -->
