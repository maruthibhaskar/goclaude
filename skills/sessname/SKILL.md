---
name: sessname
description: Use when user says "sessname {name}" to set session name, or "sessname -status" to show current session name
---

# SessName Skill Guide

## Purpose
Set or display session name marker for recall tool continuity.

## When to Use
- User says "sessname cpt" (or any name) to set session goal
- User says "sessname -status" or "sessname" alone to show current session name

## Execution

### Mode 1: Show Status (`sessname -status` or `sessname` alone)

Check environment variable `$AIA_SESSION_NAME`:

```bash
echo $AIA_SESSION_NAME
```

**Output**:
- If set: `Current session: {name}`
- If not set: `No session name set. Use: sessname {name}`

### Mode 2: Set Name (`sessname {name}`)

#### 1. Extract Name from User Input
The word after "sessname" is the session name.
Examples:
- "sessname cpt" → name = "cpt"
- "sessname sft" → name = "sft"

#### 2. Echo to Log
Output the magic string that goclaude will parse on resume:

```
AIA_SESSION_NAME='{name}'
```

**Example**: For "sessname cpt", output:
```
AIA_SESSION_NAME='cpt'
```

#### 3. Confirm to User
```
Session name set: {name}
```

## Error Handling
- No name and no -status: Show current status (same as -status)

## Why This Works
- Echo appears in session log (jsonl)
- When recall invokes goclaude --resume {id}, goclaude parses log for last AIA_SESSION_NAME
- Extracted value is exported before launching claude
- Skills like sessload can then use $AIA_SESSION_NAME
