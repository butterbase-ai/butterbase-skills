# Butterbase Skills

Claude Code plugin for [Butterbase](https://butterbase.ai) — the AI-Native Backend-as-a-Service.

This plugin gives Claude deep knowledge of Butterbase's nearly 40 MCP tools, guides you through common workflows, and auto-configures the MCP server connection.

## Installation

```bash
claude plugin marketplace add https://github.com/butterbase-ai/butterbase-skills
claude plugin install butterbase-skills@butterbase-skills
```

## Setup

Sign-in is OAuth — no API key copy-paste needed.

1. **Sign up** at [butterbase.ai](https://butterbase.ai).

2. **Install the MCP server across every detected client** with one command:
   ```bash
   npx @butterbase/cli mcp install
   ```
   This walks Claude Code, Cursor, VS Code, JetBrains, Codex, Gemini CLI, and the rest, and prints per-client OAuth follow-up hints.

3. **Trigger OAuth once per client.** In Claude Code: restart, then run `/mcp` (or `claude mcp login butterbase`). The cli's output covers every other client.

The plugin auto-loads its skills and CLAUDE.md context as soon as Claude Code starts. All tools are available immediately after consent.

## Available Skills

Skills are namespaced by the plugin name, e.g. `butterbase-skills:build-app`.

### Building blocks

| Skill | Description | Example prompts |
|-------|-------------|-----------------|
| `butterbase-skills:build-app` | Build a complete app from scratch | "Build me a blog with auth and comments" |
| `butterbase-skills:schema-design` | Design database schemas | "Design a schema for an e-commerce app" |
| `butterbase-skills:deploy-frontend` | Deploy frontends to live URLs | "Deploy my React app to production" |
| `butterbase-skills:debug-rls` | Debug Row-Level Security | "Users are seeing each other's data" |
| `butterbase-skills:function-dev` | Develop serverless functions | "Create a cron job to clean up expired sessions" |
| `butterbase-skills:auth-setup` | OAuth providers, auth hooks, JWT lifetimes, service keys | "Add Google and GitHub sign-in" |
| `butterbase-skills:storage` | File uploads/downloads, presigned URLs, storage ACLs | "Let users upload profile pictures" |
| `butterbase-skills:realtime` | WebSocket subscriptions, presence, multiplayer state | "Show new messages live without refreshing" |
| `butterbase-skills:durable-objects` | Stateful per-key actors (rooms, rate limiters, long-running agents) | "Build a multiplayer game room" |
| `butterbase-skills:ai` | AI gateway: chat, embeddings, models, defaults, BYOK, usage | "Summarize each post with an LLM" |
| `butterbase-skills:rag-dev` | Knowledge bases, document ingestion, semantic search, Q&A | "Let users ask questions about our docs" |
| `butterbase-skills:agents` | Declarative LLM/tool-graph agents, MCP servers, access controls | "Create a support agent that can look up orders" |
| `butterbase-skills:meetings` | Meeting bots that join, record, and transcribe Zoom/Meet/Teams/Webex calls | "Build a sales-call notetaker" |
| `butterbase-skills:integrations` | Third-party SaaS (email, SMS, Slack, calendar, CRM, ...) via built-in integrations | "Send a welcome email when someone signs up" |
| `butterbase-skills:payments` | Stripe Connect payments through `manage_billing` | "Charge users a monthly subscription" |
| `butterbase-skills:substrate` | Read/write the per-user agent-memory substrate | "Remember my customers across apps" |
| `butterbase-skills:migrations` | Move an app between regions, check, abort, or reverse a move | "Move my app to the EU region" |
| `butterbase-skills:templates` | Publish, browse, and clone public app templates | "Publish my app as a template" |
| `butterbase-skills:contributing` | Contribute to Butterbase | "How do I add a new MCP tool?" |

### Guided journey

`butterbase-skills:journey` takes an idea all the way to a deployed app, running these stages in order. Each stage can also be run on its own.

| Skill | Description | Example prompts |
|-------|-------------|-----------------|
| `butterbase-skills:journey` | Orchestrates the full idea → plan → build → deploy flow | "I have an idea for an app" |
| `butterbase-skills:journey-idea` | Stage 1: one-question-at-a-time idea brainstorm | "I want to build something that..." |
| `butterbase-skills:journey-plan` | Stage 2: turn the idea into a concrete Butterbase plan | "Plan out the tables and auth" |
| `butterbase-skills:journey-preflight` | Verify account, MCP connection, API key, and app ID | "Am I set up to build?" |
| `butterbase-skills:journey-docs` | Prime the relevant Butterbase docs before building | — |
| `butterbase-skills:journey-schema` | Apply the plan's tables | — |
| `butterbase-skills:journey-rls` | Install the plan's RLS policies | — |
| `butterbase-skills:journey-auth` | Configure the plan's OAuth providers and auth hooks | — |
| `butterbase-skills:journey-storage` | Configure the plan's storage buckets and ACLs | — |
| `butterbase-skills:journey-functions` | Implement and deploy the plan's functions | — |
| `butterbase-skills:journey-realtime` | Enable realtime on the plan's tables | — |
| `butterbase-skills:journey-durable` | Deploy the plan's Durable Object classes | — |
| `butterbase-skills:journey-ai` | Configure the plan's AI gateway defaults | — |
| `butterbase-skills:journey-rag` | Create the plan's RAG collections and ingest seed docs | — |
| `butterbase-skills:journey-agents` | Build and deploy the plan's agents | — |
| `butterbase-skills:journey-frontend` | Scaffold and deploy the plan's frontend | — |
| `butterbase-skills:journey-deploy` | Smoke-test the deployed app | "Check my app actually works" |
| `butterbase-skills:journey-substrate` | Optional: link the app to your substrate | — |
| `butterbase-skills:journey-templates` | Optional: publish the app as a template | — |
| `butterbase-skills:journey-submit` | Optional: submit the app to a hackathon | "Submit my app to the hackathon" |

### Slash commands

The plugin also ships 34 slash commands (in `commands/`) that invoke the skills directly, e.g. `/butterbase-skills:journey`, `/butterbase-skills:schema`, `/butterbase-skills:deploy`, `/butterbase-skills:idea`, `/butterbase-skills:submit`.

## What's Included

- **`.mcp.json`** — Auto-configures the Butterbase MCP server connection (HTTPS endpoint)
- **`CLAUDE.md`** — Always-on context: environment variables, workflows, patterns, documentation references
- **39 skills and 34 slash commands** — Guided workflows for building, deploying, debugging, and contributing, plus the end-to-end journey

## Local Development

If you're running the Butterbase monorepo locally, the MCP server URL defaults to `http://localhost:4000/mcp`. Set this in your environment:

```bash
export CONTROL_API_URL=http://localhost:4000
```

## Also Available

- **[@butterbase/sdk](https://www.npmjs.com/package/@butterbase/sdk)** — TypeScript SDK for client-side and server-side use
- **[@butterbase/cli](https://www.npmjs.com/package/@butterbase/cli)** — CLI tool for project scaffolding and backend management
- **`butterbase_docs` MCP tool** — Call with any topic (auth, storage, functions, etc.) for comprehensive reference docs

## License

MIT
