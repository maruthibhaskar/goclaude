# SessLoad Command

## Purpose
Load session summary from file and restore todos to continue work from previous conversation.

## When to Use
- User says "sessload" - read session summary and sync todos
- Session name derived from workspace (no user input needed)

---

## Execution Steps

### 1. Get Latest Session File
Use `get-current-session` tool to find latest session:
```bash
session_file=$(bin/get-current-session)
```

**Exit codes**:
- 0: Success (file path returned)
- 1: Not in workspace - inform user and STOP
- 2: No sessions exist - inform user and STOP

Read the returned file path.

### 2. Parse JSON
Read JSON file and extract:
- `workspace` - workspace name
- `goal` - session goal
- `pending_todos` - array of pending todos with context
- `completed_todos` - array of completed todos
- `session_summary` - narrative summary
- `files_modified` - files changed in session

### 3. Sync to TodoWrite

**Merge strategy**: Preserve existing todos, add missing ones

For each todo in `pending_todos`:
- If exists in TodoWrite: Keep current status (user may have made progress)
- If missing: Add with status from JSON (typically "pending")

**Rationale**: Don't overwrite progress made in current session.

### 4. Report Completion

```
✅ Loaded session: {session_file}

Workspace: {workspace}
Session goal: {goal}
Timestamp: {timestamp}

Todos loaded:
1. {Todo text} - {status in TodoWrite}
2. {Todo text} - {status in TodoWrite}
[...]

Total: {N} todos ({X} already tracked, {Y} added)

Session summary:
{First paragraph of session_summary}

Ready to continue.
```

**Note**: Include brief context from summary for orientation.

---

## Error Handling

- **Not in workspace** (exit code 1): Inform user and STOP
- **No sessions exist** (exit code 2): Inform user (this is first session), STOP
- **Malformed JSON**: Report parsing errors, show which fields failed
- **TodoWrite sync fails**: Report failed todos, continue with successful ones
