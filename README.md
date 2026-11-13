# orchestrator-implementer

A working mode for coding agents: the main agent acts as an **orchestrator** and hands self-contained tasks to an **implementer** subagent. This repository holds the instruction files for Claude Code and for Codex, and explains how to install them on Linux and Windows.

## How it works

Two pieces of text make up the mode:

- **Orchestration rules**, read by the main agent in every session. They tell it to plan and decompose the work itself, dispatch substantial tasks to the `implementer` subagent with a complete brief, review what comes back, and keep every decision (scope, git, architecture, trade-offs) for itself. Small tasks stay inline.
- **The `implementer` definition**, read by the subagent. It tells the worker to stay inside the brief, find missing facts itself but leave decisions to the orchestrator, verify before reporting, never commit unless told to, and return a fixed-format report: outcome, result, verification, issues, follow-ups.

The subagent's work happens in its own context, so the main agent's context receives only the report. A subagent starts with no knowledge of the conversation, which is why the rules insist on complete briefs.

## Files

The two harnesses read the same content in different formats, so each has its own directory.

| | Claude Code | Codex |
|---|---|---|
| Orchestration rules | [claude-code/CLAUDE.md](claude-code/CLAUDE.md) | [codex/AGENTS.md](codex/AGENTS.md) |
| Format | Markdown | Markdown |
| Installed as | `~/.claude/CLAUDE.md` | `~/.codex/AGENTS.md` |
| Implementer definition | [claude-code/agents/implementer.md](claude-code/agents/implementer.md) | [codex/agents/implementer.toml](codex/agents/implementer.toml) |
| Format | Markdown with YAML frontmatter | TOML; the prompt is the `developer_instructions` string |
| Installed as | `~/.claude/agents/implementer.md` | `~/.codex/agents/implementer.toml` |
| Model settings | `model`, `effort` | `model`, `model_reasoning_effort` |

The Codex text differs from the Claude Code text in two places, in the last two paragraphs of the orchestration rules: it refers to `AGENTS.md` where the other refers to `CLAUDE.md`, and it does not name Claude Code's `SendMessage` tool. When you change one copy, change the other.

On Windows, `~` is your user profile folder: `C:\Users\<you>`, or `$HOME` in PowerShell. Both tools use the same folder names there (`.claude`, `.codex`). If you run a tool inside WSL, follow the Linux steps inside WSL.

## Install

Clone this repository and run the commands from its root.

The orchestration rules go into a global instruction file that you may already have. The commands below create that file only when it does not exist. If it exists, open it and paste in the `## Orchestration` section from this repository.

### Claude Code on Linux

```bash
mkdir -p ~/.claude/agents
cp claude-code/agents/implementer.md ~/.claude/agents/
[ -e ~/.claude/CLAUDE.md ] && echo "CLAUDE.md exists: paste the Orchestration section in by hand" \
  || cp claude-code/CLAUDE.md ~/.claude/CLAUDE.md
```

To have `git pull` update the installed subagent, symlink it instead of copying:

```bash
ln -s "$PWD/claude-code/agents/implementer.md" ~/.claude/agents/implementer.md
```

### Claude Code on Windows

In PowerShell:

```powershell
New-Item -ItemType Directory -Force "$HOME\.claude\agents" | Out-Null
Copy-Item claude-code\agents\implementer.md "$HOME\.claude\agents\"
if (Test-Path "$HOME\.claude\CLAUDE.md") { "CLAUDE.md exists: paste the Orchestration section in by hand" }
else { Copy-Item claude-code\CLAUDE.md "$HOME\.claude\CLAUDE.md" }
```

### Codex on Linux

```bash
mkdir -p ~/.codex/agents
cp codex/agents/implementer.toml ~/.codex/agents/
[ -e ~/.codex/AGENTS.md ] && echo "AGENTS.md exists: paste the Orchestration section in by hand" \
  || cp codex/AGENTS.md ~/.codex/AGENTS.md
```

Or symlink the agent file:

```bash
ln -s "$PWD/codex/agents/implementer.toml" ~/.codex/agents/implementer.toml
```

### Codex on Windows

In PowerShell:

```powershell
New-Item -ItemType Directory -Force "$HOME\.codex\agents" | Out-Null
Copy-Item codex\agents\implementer.toml "$HOME\.codex\agents\"
if (Test-Path "$HOME\.codex\AGENTS.md") { "AGENTS.md exists: paste the Orchestration section in by hand" }
else { Copy-Item codex\AGENTS.md "$HOME\.codex\AGENTS.md" }
```

If you set `CODEX_HOME` (Codex) or `CLAUDE_CONFIG_DIR` (Claude Code), use that folder in place of `~/.codex` or `~/.claude`.

### One project only

To use the mode in a single repository, put the files in that repository instead of your home folder:

- Claude Code: `.claude/agents/implementer.md`, and the rules in `CLAUDE.md` at the repository root.
- Codex: `.codex/agents/implementer.toml`, and the rules in `AGENTS.md` at the repository root.

## Check that it works

Both tools load these files when a session starts, so open a new session after installing.

1. Ask the agent which subagents it has. `implementer` should be in the answer.
2. Give it a task with a clearly separable part, for example "find every place this repo reads an environment variable and list them". The main agent should dispatch it to `implementer` and then summarize the report it gets back.

In Codex, subagents depend on the `multi_agent` feature, which is on by default. `codex features list` shows its state.

## Adjust

- **Model and effort.** Both definitions set high reasoning effort (`effort: high` in Claude Code, `model_reasoning_effort = "high"` in Codex) and inherit the model from the parent session. The `model` line is commented out in each; uncomment it to pin one.
- **Permissions in Codex.** The agent file sets `sandbox_mode = "workspace-write"`. The current Codex documentation says a subagent runs with the sandbox and approval settings of the parent session and that an agent file cannot override them, so choose them on the parent session before delegating.
- **How much gets dispatched.** The second paragraph of the orchestration rules decides which tasks are worth a dispatch. Edit it if the main agent delegates too much or too little.

## 中文说明

这是一种 coding agent 的工作模式：主 agent 做编排者（orchestrator），负责规划、拆分、验收和所有决策；独立的、能在一份任务说明里讲清楚的工作交给 `implementer` 子 agent 执行，子 agent 只返回一份固定格式的报告。

Claude Code 和 Codex 的内容相同、格式不同，分别放在 `claude-code/` 和 `codex/` 目录：主 agent 的规则都是 Markdown（`CLAUDE.md` 和 `AGENTS.md`）；子 agent 的定义在 Claude Code 里是带 YAML frontmatter 的 Markdown，在 Codex 里是 TOML。Linux 和 Windows 的安装命令见上文 Install；Windows 上 `~` 指用户目录 `C:\Users\<用户名>`。
