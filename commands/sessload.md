# SessLoad Command

## Purpose
Load session summary from file and restore todos to continue work from previous conversation.

## When to Use
- User says "sessload" - read session summary and sync todos

---

## Execution Steps

### 1. Get Latest Session File

**Option A**: If `bin/get-current-session` tool is available:
```bash
session_file=$(bin/get-current-session)
```

**Option B**: Manual detection:
```bash
# Find workspace root
current_dir="$(pwd)"
workspace_root=""
while [[ "$current_dir" != "/" ]]; do
    if [[ -d "$current_dir/.procompact" ]]; then
        workspace_root="$current_dir"
        workspace_name=$(basename "$current_dir")
        break
    fi
    current_dir=$(dirname "$current_dir")
done

# Find latest session file
if [[ -n "$workspace_root" ]]; then
    session_file=$(ls -1 "$workspace_root/.procompact/${workspace_name}"-*.json 2>/dev/null | sort -r | head -n 1)
    # Convert to relative path
    session_file="${session_file#$workspace_root/}"
fi
```

**Error handling**:
- No workspace root found: Inform user "Not in workspace" and STOP
- No session files found: Inform user "No sessions exist" and STOP

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

For each todo in summary file:
- If exists in TodoWrite: Keep current status (user may have made progress)
- If missing: Add with status "pending"

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

- **Not in workspace**: No `.procompact` directory found - inform user and STOP
- **No sessions exist**: No JSON files found - inform user (this is first session), STOP
- **Malformed JSON**: Report parsing errors, show which fields failed
- **TodoWrite sync fails**: Report failed todos, continue with successful ones

## Tool Support

This command works with optional helper tools in `bin/`:
- `get-workspace` - Auto-detect workspace name
- `get-current-session` - Find latest session file

If tools not available, use manual detection methods shown above.
