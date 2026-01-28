---
name: core-ng-backend-coding-guidance-skill
description: Core-NG framework guidelines. Automatically activates when writing Java code in Core-NG projects, or manually via `/core-ng-backend-coding-guidance-skill`.
allowed-tools: Read, Glob
proactive: true
---

# Core-NG Backend Coding Guidance Skill

## Description

This skill integrates Core-NG framework guidelines into projects that use Core-NG. It reads the spec and shows relevant guidelines before writing Java code.

## Trigger

### Automatic Trigger (Proactive)

This skill should be **automatically activated** when:
1. The user requests to write or modify Java code
2. The project uses Core-NG framework

**How to detect Core-NG framework usage:**
- Check for `core.framework` imports in existing Java files
- Look for `Module` classes extending `core.framework.module.Module`
- Check if `build.gradle` or `build.gradle.kts` contains `core-ng` dependencies
- Look for annotations like `@Inject`, `@Property`, `@Field`, `@Collection` from `core.framework` package

When Core-NG is detected, Claude MUST read the specification before writing any Java code.

### Manual Trigger

- `/core-ng-backend-coding-guidance-skill` - Read spec and show relevant guidelines

## How It Works

When `/core-ng-backend-coding-guidance-skill` is triggered, Claude MUST:

1. **Read the specification file**: `~/.claude/skills/core-ng-backend-coding-guidance-skill/assets/spec.md`

2. **Analyze the current context**: Based on the conversation history, identify what type of Java code will be written (e.g., API interface, MongoDB entity, service, Kafka handler, etc.)

3. **Extract and display relevant guidelines**: Show only the sections of the spec that are relevant to the current task. For example:
   - If writing an API response class → show "Interface Class" section
   - If writing a MongoDB entity → show "MongoDB" section
   - If writing a service → show "Dependency Injection" section
   - If handling exceptions → show "Exception Handling" section

4. **Confirm readiness**: After showing the guidelines, confirm that you're ready to write code following these conventions.

## Assets

- `assets/spec.md` - The Core-NG framework specification document containing detailed guidelines for:
  - Dependency injection patterns
  - MongoDB usage conventions
  - Kafka integration
  - WebService API design
  - Exception handling
  - And more framework-specific requirements

## Usage

1. Navigate to a project that uses Core-NG framework
2. Before writing any Java code, run `/core-ng-backend-coding-guidance-skill`
3. The skill will read the specification and display relevant guidelines based on the current task
4. Proceed with code generation following the displayed conventions
