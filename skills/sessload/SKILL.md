---
name: sessload
description: Use when user says "sessload" to load session summary and restore todos
---

# SessLoad Skill Guide

## Purpose
Load session summary from file and restore todos to continue work from previous conversation.

## When to Use
- User says "sessload" - read session summary and sync todos

---

## Execution Steps

### 1. Check Environment
```bash
echo $AIA_SESSION_NAME
```
If set, use as `{sessionname}`. If not set, error: "AIA_SESSION_NAME not set, and STOP. Cannot load session summary."

### 2. Find and Read Summary File
Find session files matching `.procompact/{sessionname}_*.md`:
```bash
FILES=$(ls -1 .procompact/{sessionname}_*.md 2>/dev/null)
COUNT=$(echo "$FILES" | grep -c . || echo 0)
LATEST=$(echo "$FILES" | sort | tail -1)
```

- If no files found (COUNT=0): Error "No session summary found for '{sessionname}'" and STOP
- If multiple files found: Automatically use LATEST file (don't ask user)
- Report which file was loaded if multiple existed

Read the found file.

### 3. Parse Todos
Extract from "Pending Todos" section:
- Todo text
- Goal
- Context

### 4. Sync to TodoWrite

**Merge strategy**: Preserve existing todos, add missing ones

For each todo in summary file:
- If exists in TodoWrite: Keep current status (user may have made progress)
- If missing: Add with status "pending"

**Rationale**: Don't overwrite progress made in current session.

### 5. Report Completion

```
✅ Loaded session summary: .procompact/{sessionname}_{TIMESTAMP}.md
{If multiple files existed: "(Selected latest of {COUNT} files)"}

Session goal: {goal}

Todos loaded:
1. {Todo text} - {status in TodoWrite}
2. {Todo text} - {status in TodoWrite}
[...]

Total: {N} todos ({X} already tracked, {Y} added)

Ready to continue.
```

**Note**: Include brief context from summary if helpful for orientation.

---

## Error Handling

- **AIA_SESSION_NAME missing**: Error and stop
- **File missing**: STOP and ask user for direction (don't assume sesssave needed)
- **Malformed file**: Report parsing errors, show which todos failed
- **TodoWrite sync fails**: Report failed todos, continue with successful ones
