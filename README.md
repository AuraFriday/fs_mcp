# FS — Consolidated Filesystem, Search & Diagnostics Toolbox

**One tool. Twelve operations. Everything your AI needs to read, write, search, and manage files.**

[![License](https://img.shields.io/badge/license-Proprietary-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.9%2B-blue.svg)](https://www.python.org/)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20Linux%20%7C%20macOS-lightgrey.svg)](https://github.com/AuraFriday/mcp-link-server)

---

## What This Means For You

### 1. Your AI Has Full File Access

Read any file (text, images, PDFs). Write or overwrite files atomically. Edit files with precise string replacement. Delete files safely. List directories. Search across your codebase by name, content, or semantics. All from one tool, with workspace containment and protected paths.

### 2. One Tool Instead of Ten

The `fs` tool consolidates what used to be ten standalone tools into a single, efficient entry point. Your AI loads one readme, gets one unlock token, and has access to everything — no juggling ten separate tool names.

### 3. Progressive Disclosure

The readme gives a compact overview. Each operation has its own full manual (via `operation: "manual", topic: "<operation>"`), so your AI only loads the documentation it needs for the current task. This saves context and makes the tool fast to use.

---

## Operations At a Glance

| Operation | Description |
|-----------|-------------|
| `read` | Read file contents (text with line numbers; images; PDF text extraction) |
| `write` | Create or overwrite a file (atomic; optional `overwrite=false` / `backup=true`) |
| `str_replace` | Replace an exact string in a file (atomic; optional `replace_all`) |
| `delete` | Delete a single file (idempotent; optional `trash=true` for recoverable delete) |
| `list` | List a directory (optional dot entries, ignore globs, per-entry details) |
| `glob` | Find files by glob name pattern (recursive; newest first) |
| `grep` | Regex content search across files (content / files_with_matches / count modes) |
| `code_search` | Keyword-ranked code search across workspace files |
| `web_search` | Web search via DuckDuckGo (results are untrusted content) |
| `read_lints` | Run available linters on files/directories and report diagnostics |
| `transfer` | Push files/folders to an admitted den peer over iroh (recursive; sha256-verified; resumes) |

Plus:
| `readme` | Compact overview + unlock token (no token required) |
| `manual` | Full manual for one operation (no token required) |

---

## How It Works

### Get Started
```json
{"input": {"operation": "readme"}}
```
Returns: every operation with a one-line summary, this tool's unlock token, and how to fetch manuals.

### Read a Full Manual
```json
{"input": {"operation": "manual", "topic": "grep"}}
```
Returns: the complete parameter reference for that one operation.

### Use Any Operation
```json
{"input": {"operation": "read", "path": "/home/user/project/main.py", "tool_unlock_token": "..."}}
```

---

## Operation Details

### read
Read text files (with line numbers), binary images (returned as base64), or PDFs (text extraction). Supports offset/limit for large files.

### write
Atomic file creation or overwrite. Supports:
- `overwrite: false` — refuse if the file already exists
- `backup: true` — create a `.bak` copy before overwriting

### str_replace
Find-and-replace with atomic writes. The `old_string` must appear exactly once (unless `replace_all: true`). This is the primary editing operation for AI code modifications.

### delete
Remove a single file. Idempotent (no error if already gone). Optional `trash: true` moves to a recoverable location instead of permanent deletion.

### list
Directory listing with optional:
- Dot-file inclusion
- Ignore patterns (gitignore-style globs)
- Per-entry metadata (size, modified time, type)

### glob
Find files by name pattern (e.g. `**/*.py`, `src/**/test_*.ts`). Results sorted newest-first. Recursive by default.

### grep
Full regex content search powered by ripgrep semantics:
- `content` mode: shows matching lines with context
- `files_with_matches` mode: just file paths
- `count` mode: match counts per file
- Supports `-A`, `-B`, `-C` context lines, case-insensitive mode

### code_search
Keyword-ranked code search across workspace files. Not embedding-based — uses term frequency and file relevance scoring. Good for finding specific symbols, function names, and patterns.

### web_search
Search the web via DuckDuckGo. Returns summarized results with URLs. Results are untrusted external content — always verify before acting on them.

### read_lints
Run available linters (language-specific) on files or directories and report diagnostics. Useful for checking your edits introduced no errors.

### transfer
Push files or entire folder trees to another device via den (iroh QUIC). Features:
- Recursive directory transfer
- SHA-256 verification of every file
- Resume support for interrupted transfers
- Requires the target device to be an admitted den peer

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        fs tool (single entry point)             │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Routing table: operation → delegate module               │   │
│  └──────────────────────────────────────────────────────────┘   │
│       │         │         │         │         │         │       │
│       ▼         ▼         ▼         ▼         ▼         ▼       │
│  file_read  file_write  file_glob  file_grep  ...  file_transfer│
│  (original implementations, all behavior preserved)             │
└─────────────────────────────────────────────────────────────────┘
```

`fs` does NOT reimplement any file logic. Each operation delegates to the original module's handler with that module's own unlock token injected. The delegate modules keep all their validation (workspace containment, protected paths, size caps, time budgets).

---

## Security

- **Workspace Containment**: operations are constrained to the configured workspace paths. Attempts to read/write outside the workspace are rejected.
- **Protected Paths**: certain system files and directories are off-limits regardless of workspace setting.
- **Size Caps**: large file reads are bounded; extremely large writes are refused.
- **Time Budgets**: search operations have timeouts to prevent runaway scans.
- **Atomic Writes**: `write` and `str_replace` use atomic rename-into-place, so a crash mid-write never leaves a corrupted file.

---

## Relationship to Cursor IDE Integration

When running inside Cursor (or VS Code with the Aura Friday extension), `fs` provides the same operations that IDE-native file tools use. If IDE-duplicate tools are disabled in settings, the standalone `file_read`, `file_write` etc. names disappear from the tool list and only `fs` remains — a single clean entry point.

---

> 🧠 **For AI agents:** Start with `{"input":{"operation":"readme"}}` to get the overview and your unlock token. Then use `{"input":{"operation":"manual","topic":"<op>"}}` to load only the manual you need. This keeps your context lean while giving you full filesystem access.
