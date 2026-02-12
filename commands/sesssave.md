# SessSave Command

## Purpose
Save compact session summary with pending todos for resuming work in new conversation.

## When to Use
- User says "sesssave" - generate/update session summary file

---

## Execution Steps

### 1. Get Workspace Name
Determine workspace name from current directory. Options:

**Option A**: If `bin/get-workspace` tool is available:
```bash
workspace_name=$(bin/get-workspace)
```

**Option B**: Manual detection (find `.procompact` marker in parent tree):
```bash
current_dir="$(pwd)"
while [[ "$current_dir" != "/" ]]; do
    if [[ -d "$current_dir/.procompact" ]]; then
        workspace_name=$(basename "$current_dir")
        break
    fi
    current_dir=$(dirname "$current_dir")
done
```

If no `.procompact` directory found, error: "Not in workspace"

### 2. Get All Todos
Query TodoWrite tool for all todos:
- Pending/in_progress: Will be listed in "Pending Todos" section
- Completed: Will be listed in "Completed Work" section (for session summary)

### 3. Determine Session Goal
Infer from context using this priority:
1. First pending todo's goal/context
2. Pattern of completed work
3. If cannot infer: Use "Session work in progress" as placeholder
**Never interactively ask user for goal.**

### 4. Write Summary File
Generate filename: `{workspace-name}-{YYYYMMDD-HHMMSS}.json`

**Option A**: If `bin/get-session-filename` tool is available:
```bash
filename=$(bin/get-session-filename)
```

**Option B**: Manual generation:
```bash
timestamp=$(date +"%Y%m%d-%H%M%S")
filename="${workspace_name}-${timestamp}.json"
```

Write to `.procompact/$filename` with JSON session data:

```json
{
  "workspace": "{workspace_name}",
  "timestamp": "{YYYYMMDD-HHMMSS}",
  "goal": "{session goal}",
  "pending_todos": [...],
  "completed_todos": [...],
  "session_summary": "...",
  "files_modified": [...]
}
```

**JSON Structure**:
```json
{
  "workspace": "workspace-name",
  "timestamp": "YYYYMMDD-HHMMSS",
  "goal": "Brief session goal description",
  "pending_todos": [
    {
      "content": "Todo text",
      "status": "pending|in_progress",
      "context": "What we discussed, learned, decided so far"
    }
  ],
  "completed_todos": [
    {
      "content": "Todo text",
      "outcome": "What was accomplished"
    }
  ],
  "session_summary": "1-3 paragraph summary (max 500 words). Focus on WHY decisions were made.",
  "files_modified": [
    {
      "path": "path/to/file.ext",
      "changes": "Brief description"
    }
  ],
  "production_status": {
    "branch": "branch name",
    "last_commit": "commit message if recent",
    "build_status": "passing/failing if relevant"
  }
}
```

**Key principles**:
- Pending todos include full context for resumption
- Session Summary captures narrative arc and decisions (helps bypass autocompaction)
- Completed work provides closure and context
- 500 word limit for summary section keeps it focused
- All information embedded for standalone resumption

### 5. No Cleanup Required
Old sessions are kept for reference. No deletion of previous session files.
Users can manually clean up `.procompact/` directory if needed.

### 6. Report Completion

Inform user with concise summary:

```
✅ Session saved: .procompact/{workspace-name}-{YYYYMMDD-HHMMSS}.json

Workspace: {workspace_name}
Goal: {goal}
Pending todos: {N}
Completed work: {M} tasks

To resume: Start new session, say "sessload"
```

---

## Error Handling

- **Not in workspace**: No `.procompact` directory found - inform user
- **No pending todos**: Save file anyway with empty pending_todos array
- **File write fails**: Report error with path

## Tool Support

This command works with optional helper tools in `bin/`:
- `get-workspace` - Auto-detect workspace name
- `get-session-filename` - Generate standardized filename

If tools not available, use manual detection methods shown above.
