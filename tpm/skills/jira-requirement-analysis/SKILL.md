---
name: jira-requirement-analysis
description: Analyze Jira tickets and post structured requirement analysis as comments. Use when user provides a Jira ticket URL for requirement analysis, wants to understand implementation scope, or asks to analyze a ticket before development. Triggers on phrases like "analyze Jira ticket", "requirement analysis for TICKET-XXX", or "post analysis to Jira".
---

# Jira Requirement Analysis

Analyze Jira tickets, search the codebase for relevant files, and auto-post structured requirement analysis as a Jira comment.

## Pre-checks (STOP if failed)

| Condition | Action |
|-----------|--------|
| Jira ticket unclear/incomplete/ambiguous | Invoke brainstorm skill to reason, infer, and structure the requirement |
| Code repository or location unclear/missing | STOP and ask user for clarification |

## Workflow

1. **Fetch Jira ticket** - Get details, attachments, linked tickets using MCP tools
2. **Search codebase** - Find all relevant files, models, migrations, APIs
3. **Analyze** - Understand current implementation and required changes
4. **Generate analysis** - Produce structured requirement analysis (format below)
5. **Post to Jira** - Add comment to the ticket automatically
6. **Tag ticket** - Add label `AI-RA` to the ticket

## Output Format (Single Jira Comment)

```markdown
## Requirement Analysis

### 1. Core Requirement (What & Why)
**What:** [One sentence describing the change]
**Why:** [Business reason / context]

### 2. Key Impacts
| Area | Impact |
|------|--------|
| **Scope** | [Backend/Frontend/Both] |
| **Data** | [DB changes, migrations] |
| **API** | [Endpoint changes] |
| **UI** | [Frontend changes] |
| **Logic** | [Business logic changes] |
| **Prerequisite** | [Dependencies, validations] |

### 3. Implementation Approach (Pseudocode)
[Brief pseudocode - NO actual implementation]
[MAY quote existing code snippets]

### 4. Code References
- `file.java:line` - description
- `file.ts:line` - description

### 5. Related Tickets
- **TICKET-XXX** (Status): Description
```

## Constraints

- Be extremely concise
- No unnecessary explanation
- Use markdown tables and code blocks
- Auto-post to Jira after analysis
