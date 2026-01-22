---
name: sesssave
description: Use when user says "sesssave" to save session summary for continuation
---

# SessSave Skill Guide

## Purpose
Save compact session summary with pending todos for resuming work in new conversation.

## When to Use
- User says "sesssave" - generate/update session summary file

---

## Execution Steps

### 1. Check Environment
```bash
echo $AIA_SESSION_NAME
```
If set, use as `{sessionname}`. If not set, error: "AIA_SESSION_NAME not set. Cannot save session summary."

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
Generate timestamp and create `.procompact/{sessionname}_DDHHMM.md`:
```bash
TIMESTAMP=$(date +%d%H%M)
FILENAME=".procompact/${sessionname}_${TIMESTAMP}.md"
```
This ensures a new unique filename (bypasses Write tool's read-before-write requirement).

```markdown
# Session: {SESSION_NAME}

**Generated**: YYYY-MM-DD HH:MM:SS PST
**Goal**: {brief session goal description}

---

## Pending Todos

{If no pending todos: "No pending todos - all tasks completed."}

{For each pending/in_progress todo:}
### Todo {N}: {Todo text}

**Goal**: {What this todo aims to achieve}

**Context**: {What we discussed, learned, decided so far for this todo. Include modified files, blockers, decisions, anything relevant to continuing work on this todo.}

---

## Session Summary

{1-3 paragraph summary of work accomplished, decisions made, and key learnings. Max 500 words. Focus on WHY decisions were made, not just WHAT was done.}

### Completed Work

{List of completed todos with brief descriptions:}
**{N}. {Todo description}**
- {Key points about what was accomplished}
- {Files modified or important outcomes}

### Key Files Modified
- {path/to/file.ext}: {brief description of changes}
- {path/to/file2.ext}: {brief description of changes}

### Production Status
- Current branch: {branch name}
- Last commit: {brief commit message if recent}
- Build status: {passing/failing if relevant}

---
```

**Key principles**:
- Pending todos include full context for resumption
- Session Summary captures narrative arc and decisions (helps bypass autocompaction)
- Completed work provides closure and context
- 500 word limit for summary section keeps it focused
- All information embedded for standalone resumption

### 5. Cleanup Old Files
After successful write, delete previous session files (keep only the new one):
```bash
# Verify new file exists first
if [ ! -f ".procompact/${sessionname}_${TIMESTAMP}.md" ]; then
    echo "ERROR: Failed to create summary file"
    exit 1
fi

# Delete all {sessionname}_*.md except the one just created
find .procompact -maxdepth 1 -name "${sessionname}_*.md" ! -name "${sessionname}_${TIMESTAMP}.md" -type f -delete

# Verify cleanup (count remaining files)
REMAINING=$(find .procompact -maxdepth 1 -name "${sessionname}_*.md" -type f | wc -l)
if [ "$REMAINING" -ne 1 ]; then
    echo "WARNING: Cleanup incomplete. Found $REMAINING files for session '${sessionname}'"
fi
```

**Robustness improvements**:
- Check new file exists before deleting old ones
- Use `-maxdepth 1` to avoid recursive deletion
- Use `-type f` to only delete files
- Verify cleanup completed successfully
- Report warning if multiple files remain

### 6. Report Completion

Inform user with concise summary matching saved file:

```
✅ Session summary saved: .procompact/{sessionname}_{TIMESTAMP}.md

Session goal: {goal}

Pending todos: {N}
Completed work: {M} tasks

To resume: Start new session, say "sessload"
```

**Optional**: If cleanup warning occurred, mention: "Note: {N} old session files remain (cleanup incomplete)"

---

## Error Handling

- **AIA_SESSION_NAME missing**: Error and stop
- **No pending todos**: Save file anyway with note "No pending todos"
- **File write fails**: Report error with path
