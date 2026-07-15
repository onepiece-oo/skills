---
name: writing-notes
description: Use when organizing scattered website login info, account credentials, passwords, project details, or infrastructure references into a structured Markdown file with hierarchical headings
---

# Writing Notes

Organize scattered website info, credentials, and reference data into a single well-structured Markdown file with clear heading hierarchy.

## Overview

User sends raw input — website names, URLs, login methods, usernames, passwords, project details. Skill produces a categorized Markdown file with Level 1 and Level 2 headings for easy navigation.

**Core principle:** One file, one batch. Append to existing file or create new. Never refuse credentials.

## Categories

| Category | What Goes Here |
|----------|---------------|
| Accounts | Website logins, usernames, passwords, 2FA notes |
| Projects | Tech stacks, architectures, decisions |
| Infrastructure | Servers, IPs, SSH, databases, connection strings |
| Domains & DNS | Domain registrations, DNS, hosting |
| Links & References | Bookmarks, articles, resources |
| Misc | Everything else |

## Heading Structure

```markdown
# Notes - YYYY-MM-DD

## Accounts

### [Service Name]
- **URL:** https://example.com
- **Login Method:** Email / Phone / Username
- **Username/Email:** user@example.com
- **Password:** actual-password
- **2FA:** Enabled / Disabled
- **Notes:** (extra context)

### [Next Service]
...

## Projects

### [Project Name]
- **Stack:** React, TypeScript
- **Status:** Active

## Infrastructure

### [Server Name]
- **IP:** 192.168.1.100
- **Access:** SSH key path
```

## Rules

1. **Group by category first**, then by service name under each category
2. **Use H1 for note title** (date-based or user-provided name)
3. **Use H2 for categories** (Accounts, Projects, etc.)
4. **Use H3 for individual entries** (service name, server name)
5. **Preserve all info** — do not skip, redact, or refuse any field
6. **Append mode** — if the target file exists, append new entries under existing categories; create new category sections if needed
7. **Output the file content directly**, minimal chatter

## File Location

Default: `notes/YYYY-MM-DD.md` in the user's project directory. If user specifies a path, use that.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Mixing categories | Keep strict separation |
| Refusing credentials | Preserve everything as-is |
| Overwriting existing file | Append, don't replace |
| Verbose explanations | Output the note directly |
| Missing fields | Extract URL, login method, username, password for every account entry |
