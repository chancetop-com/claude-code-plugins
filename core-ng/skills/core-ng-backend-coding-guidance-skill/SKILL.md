---
name: core-ng-backend-coding-guidance-skill
description: Core-NG framework guidelines. Automatically activates when writing Java code in Core-NG projects, or manually via `/core-ng-backend-coding-guidance-skill`.
allowed-tools: Read, Glob
proactive: true
---

# Core-NG Backend Coding Guidance Skill

## Description

This skill integrates Core-NG framework guidelines into projects that use Core-NG. It reads the specification and ensures code follows framework conventions.

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

## Assets

Single specification file with two parts:

**`assets/spec.md`** (~700 lines)
- **Part 1: Core Rules & Patterns** (Sections 1-6) - Essential rules, always read
- **Part 2: API Reference** (Sections 7-16) - Detailed API, read when needed

## How It Works

When this skill is triggered, Claude MUST:

### Step 1: Read Part 1 (Always)

Read `assets/spec.md` (relative to this skill's directory) sections 1-6:
- Section 1: Framework Constraints
- Section 2: Essential Patterns
- Section 3: Common Mistakes & Fixes
- Section 4: Exception Handling
- Section 5: Annotation Summary
- Section 6: Design Principles

### Step 2: Read Part 2 Sections (As Needed)

Based on the current task, read relevant API Reference sections:

| Task Type | Read Section |
|-----------|--------------|
| Module setup | Section 7 (Module System) |
| MongoDB entity/query | Section 8 (MongoDB Reference) |
| SQL entity/query | Section 9 (Database Reference) |
| Kafka publisher/handler | Section 10 (Kafka Reference) |
| WebService/Controller | Section 11 (HTTP Reference) |
| Redis/Cache | Section 12 (Redis/Caching Reference) |
| Scheduled job | Section 13 (Scheduler Reference) |
| WebSocket/SSE | Section 14 (WebSocket/SSE Reference) |
| Logging/monitoring | Section 15 (Logging Reference) |
| Test setup | Section 16 (Testing Reference) |

### Step 3: Impact Analysis (MANDATORY - Before Writing Any Code)

**CRITICAL: You MUST complete this analysis before writing any code. Do NOT skip this step.**

#### 3.1 Trace Data Origin
Ask yourself:
- Where does this class/field come from?
- Which layer does it belong to? (MongoDB domain? API response? Request? Kafka message?)

#### 3.2 Map the Change Chain
Draw out the complete data flow and identify ALL layers affected:

```
[MongoDB Domain] → [Module Registration] → [Service Layer] → [API Response]
       ↑                   ↑                     ↑                ↑
  Inner class?        view() needed?        Mapper needed?    Inner class?
  @Field annotations  bind() needed?        Builder needed?   @Property annotations
```

#### 3.3 Create File Checklist
List ALL files that need modification BEFORE writing any code:

**For MongoDB inner class changes:**
- [ ] Domain class (with `@Field` annotations)
- [ ] ALL MongoModule files that use this entity → search `view(ParentClass.` to find them
- [ ] Service layer (mapper/builder)
- [ ] Response class (with `@Property` annotations)

**For new Service/Handler:**
- [ ] Service class
- [ ] Module registration (`bind()`)
- [ ] For Kafka: named handler class (no lambdas)
- [ ] For Jobs: named job class (no lambdas)

**For API changes:**
- [ ] Interface definition (in `*-interface` module)
- [ ] Implementation class
- [ ] Request/Response beans with correct annotations

#### 3.4 Compliance Check
For each file in your checklist, verify against spec.md:
- [ ] Correct annotation system? (`@Field` for MongoDB, `@Property` for API, `@Column` for SQL)
- [ ] Inner class registered? (`view()` for MongoDB inner classes)
- [ ] Service bound? (`bind()` in Module)
- [ ] Single constructor only?
- [ ] Field injection with `@Inject`?

**Output your analysis before proceeding to code changes.**

### Step 4: Apply Guidelines

After completing impact analysis, apply the guidelines when writing code:
- Follow framework constraints strictly
- Use correct annotations for each layer
- Handle exceptions properly
- Register all required classes
- **Modify ALL files identified in Step 3**

## Document Structure

```
spec.md
├── Part 1: Core Rules & Patterns (Always Read)
│   ├── 1. Framework Constraints
│   ├── 2. Essential Patterns
│   ├── 3. Common Mistakes & Fixes
│   ├── 4. Exception Handling
│   ├── 5. Annotation Summary
│   └── 6. Design Principles
│
└── Part 2: API Reference (Read When Needed)
    ├── 7. Module System Reference
    ├── 8. MongoDB Reference
    ├── 9. Relational Database Reference
    ├── 10. Kafka Reference
    ├── 11. HTTP/WebService Reference
    ├── 12. Redis and Caching Reference
    ├── 13. Scheduler Reference
    ├── 14. WebSocket and SSE Reference
    ├── 15. Logging and Monitoring Reference
    └── 16. Testing Reference
```

## Usage

1. Navigate to a project that uses Core-NG framework
2. Before writing any Java code, the skill activates automatically (or run `/core-ng-backend-coding-guidance-skill`)
3. The skill reads Part 1, then relevant Part 2 sections based on task
4. Proceed with code generation following the conventions
