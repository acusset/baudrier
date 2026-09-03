---
# ─── Required ────────────────────────────────────────────────────────────────
name: code-reviewer
# Unique identifier. Lowercase letters, numbers, hyphens only.
description: Reviews code for quality and best practices. Use after writing or modifying code.
# When Claude should delegate to this subagent. Any text.
# Write it as a trigger ("Use when…", "Use after…") so delegation is reliable.
# ─── Optional ────────────────────────────────────────────────────────────────
tools: Read, Glob, Grep
# Allowlist of tools. Omit to inherit every tool available to subagents.
# Exact names (`Read, Glob, Grep`), `Tool(type)` restriction syntax
# (e.g. `Agent(worker, researcher)`), or MCP patterns (`mcp__github`, `mcp__github__*`).

disallowedTools: mcp__github
# Denylist. Applied first, then `tools` is resolved against what remains.
# A tool named in both lists is removed. `mcp__*` removes every MCP tool.

model: sonnet
# `sonnet` | `opus` | `haiku` | `fable` | a full model ID (e.g. claude-opus-5) | `inherit`.
# Omit to let Claude Code pick via the subagent model order.
# `inherit` uses the main conversation's model.
# `CLAUDE_CODE_SUBAGENT_MODEL` sets a default; this field takes precedence over it.

permissionMode: default
# `default` | `acceptEdits` | `auto` | `dontAsk` | `bypassPermissions` | `plan` | `manual` (v2.1.200+).
# Ignored for plugin subagents. A parent in `bypassPermissions` or `acceptEdits`
# takes precedence and cannot be overridden here.

maxTurns: 30
# Positive integer. Max agentic turns before stopping.
# On reaching the limit, output is marked partial (v2.1.246+); Claude can resume.

effort: medium
# `low` | `medium` | `high` | `xhigh` | `max`. Overrides session effort while active.
# Defaults to inheriting from the session. Available levels depend on the model.

skills: compound-pattern, slice-large-components
# Skills preloaded into context at startup (full content injected).
# The subagent can still invoke unlisted skills via the Skill tool.
# Cannot preload skills with `disable-model-invocation: true`.

memory: project
# `user` | `project` | `local`. Enables cross-session learning.
#   user    → ~/.claude/agent-memory/<name>/
#   project → .claude/agent-memory/<name>/
#   local   → .claude/agent-memory-local/<name>/

color: blue
# `red` | `blue` | `green` | `yellow` | `purple` | `orange` | `pink` | `cyan`.
# Display color in the task list and transcript.

background: false
# `true` | `false`. Keep the subagent in the background even when Claude asks for foreground.
# Under fork mode, spawned subagents already run in the background by default.

isolation: worktree
# `worktree` — run in an isolated git worktree branched from the default branch.
# Auto-cleaned if no changes are made.

initialPrompt: Review the current diff.
# Auto-submitted as the first user turn when this agent runs as the main session
# (via `--agent` or the `agent` setting). Prepended to any user-provided prompt.
# Supports commands and skills.

mcpServers:
  # Server names, or inline definitions scoped to this subagent only.
  # Ignored for plugin subagents.
  - github

hooks:
  # Lifecycle hooks that run only while this subagent is active.
  # Ignored for plugin subagents. Project-level agents require workspace trust.

experimental:
  cacheTtl: 1h
  # `5m` or `1h` — prompt cache lifetime. Any other value is ignored.
  # `1h` is ignored while running on credits. Must be nested under `experimental`. v2.1.248+
---

You are a code reviewer. When invoked, analyze the code and provide
specific, actionable feedback on quality, security, and best practices.

<!--
Everything after the closing `---` is the system prompt, in Markdown.
The subagent receives only this body plus basic environment details (e.g. the
working directory) — not the Claude Code system prompt.

Claude Code SKIPS a file (treats it as documentation or logs an error) when:
  - it has no `name`
  - the opening `---` is not on the first line
  - `name` starts with `-` or contains `:`
  - `name` is present but `description` is missing
  - the YAML fails to parse
-->
