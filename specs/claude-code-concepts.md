# Claude Code Concepts: Spec Sheet

A reference guide explaining the key extensibility and automation concepts in Claude Code.

---

## Slash Commands

**What they are:** User-invocable shortcuts that trigger predefined prompts or workflows within a Claude Code session.

**How they work:**
- Invoked by typing `/command-name` in the chat input
- Can be built-in (e.g., `/help`, `/clear`, `/compact`) or user-defined
- User-defined slash commands are stored as Markdown files in `.claude/commands/` (project-level) or `~/.claude/commands/` (global)
- The filename becomes the command name; the file content becomes the prompt sent to Claude

**Example:**
```
# File: .claude/commands/review-pr.md
Review this pull request for bugs, security issues, and style problems.
```
Invoked as `/review-pr`

**Key traits:**
- Synchronous — Claude responds immediately in the current session
- Human-initiated — a user must type the command
- Scoped to the current conversation context

---

## Hooks

**What they are:** Shell commands that Claude Code executes automatically in response to specific lifecycle events.

**How they work:**
- Configured in `settings.json` under the `hooks` key
- Trigger on events like `PreToolUse`, `PostToolUse`, `Stop`, `Notification`, etc.
- Run as shell commands outside of Claude's context — Claude does not execute them, the harness does
- Can block tool use (by exiting non-zero on `PreToolUse`) or run fire-and-forget side effects

**Example:**
```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [{ "type": "command", "command": "echo 'Claude finished' | notify-send" }]
      }
    ]
  }
}
```

**Key traits:**
- Event-driven — fire automatically without user action
- Shell-level — run arbitrary system commands
- Can intercept and gate tool execution (`PreToolUse`)
- Cannot call back into Claude — they are one-way side effects

---

## Plugins

**What they are:** In the Claude Code context, "plugins" typically refers to IDE extensions or integrations that embed Claude Code into a development environment.

**How they work:**
- Available for VS Code, JetBrains, and other IDEs
- Provide a native UI panel for interacting with Claude Code from within the editor
- Share the same underlying Claude Code engine and settings

**Key traits:**
- UI/environment integrations, not behavior extensions
- Do not add new Claude capabilities — they surface existing ones in a new interface
- Installed via the IDE's extension marketplace

> **Note:** "Plugin" is not a first-class Claude Code extensibility primitive the way hooks, skills, or MCPs are. If you encounter the term in other contexts, it may refer loosely to any of those.

---

## Skills

**What they are:** Curated, reusable prompt templates that Claude Code loads and can invoke on the user's behalf — similar to slash commands but typically more sophisticated and managed by the platform.

**How they work:**
- Appear as available actions listed in system-reminder messages during a session
- Invoked via the `Skill` tool internally, or by the user typing `/skill-name`
- Can encapsulate multi-step workflows (e.g., `/commit`, `/review-pr`, `/schedule`)
- The skill content expands into a full prompt that guides Claude's behavior for that task

**Example skills:**
- `/commit` — stages, drafts a commit message, and commits changes
- `/simplify` — reviews changed code for quality and cleans it up
- `/schedule` — creates a recurring scheduled agent trigger

**Key traits:**
- Higher-level than raw slash commands — often orchestrate multiple tool calls
- Managed by the Claude Code platform or workspace administrators
- Reusable across projects without per-project configuration

---

## MCPs (Model Context Protocol Servers)

**What they are:** External servers that expose tools, resources, and prompts to Claude via the Model Context Protocol — Claude's standardized interface for connecting to external systems.

**How they work:**
- Run as separate processes (local or remote) that communicate with Claude Code over stdio or HTTP/SSE
- Configured in `settings.json` under `mcpServers`
- Each MCP server exposes a set of **tools** (callable functions), **resources** (readable data), and/or **prompts** (reusable templates)
- Claude can call MCP tools the same way it calls built-in tools like `Read` or `Bash`

**Example configuration:**
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "ghp_..." }
    }
  }
}
```

**Key traits:**
- The most powerful extensibility primitive — adds net-new capabilities to Claude
- Language-agnostic — MCP servers can be written in any language
- Sandboxed — Claude calls MCP tools; MCP servers do not control Claude
- Composable — multiple MCP servers can be active simultaneously
- Examples: GitHub integration, database access, web search, custom APIs

---

## Comparison Summary

| Concept | Who triggers it | Where it runs | Primary use case |
|---|---|---|---|
| **Slash Command** | User | Claude (as a prompt) | Reusable prompt shortcuts |
| **Hook** | Lifecycle event | Shell (outside Claude) | Automated side effects, gating |
| **Plugin** | N/A (IDE integration) | IDE environment | UI surface for Claude Code |
| **Skill** | User or Claude | Claude (as a prompt) | Curated multi-step workflows |
| **MCP** | Claude (via tool call) | External process | Adding new capabilities/tools |
