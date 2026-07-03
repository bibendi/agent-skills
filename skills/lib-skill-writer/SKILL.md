---
name: lib-skill-writer
description: Teaches the agent how to architect and write multi-file Agent Skills for large libraries (gems, packages, frameworks), and how to keep them up to date.
---
# Library Skill Writer

## Context & Purpose
Use this skill when generating or maintaining AI agent documentation for large, complex, multi-module libraries. The agent does not need a mirror of the codebase; it needs a curated map of how the library behaves at runtime, how to bootstrap and migrate it, which real-world tasks it solves, and how to recover when things go wrong.

This skill is used both for the first-time generation of a skill and for incremental updates when the library changes.

## Target Directory Architecture
Every generated skill must follow this file tree and be created inside the current project:

```text
./skills/{skill-name}/
├── SKILL.md
└── references/
    ├── architecture.md
    ├── initialization.md
    ├── migration.md
    ├── recipes/
    │   ├── {use-case-1}.md
    │   ├── {use-case-2}.md
    │   └── ...
    └── troubleshooting.md
```

Always place the generated skill at `./skills/{skill-name}/` relative to the project root. Do not write skill files to temporary directories, system paths, or locations outside the current project.

## 1. Root File (SKILL.md)

SKILL.md serves as a router. It contains no implementation details, no class lists, and no method signatures. Keep it short enough to scan instantly.

### Routing Map
- Runtime behavior, lifecycle, state, resources, concurrency: `references/architecture.md`
- Bootstrap and configuration: `references/initialization.md`
- Version upgrades and deprecations: `references/migration.md`
- Concrete use cases and tasks: `references/recipes/{use-case}.md`
- Errors and debugging: `references/troubleshooting.md`

### How to Choose a Reference
Match the user's request to the file name. If no exact match exists, read the closest reference and `architecture.md` before acting. Never guess based on a module name alone.

### Reading Order
1. `architecture.md` — understand the runtime model and constraints before touching code.
2. `initialization.md` — if the task involves setup or configuration.
3. `recipes/{use-case}.md` — the specific task the user wants to perform.
4. `migration.md` — when migrating code or handling deprecations.
5. `troubleshooting.md` — only when diagnosing failures.

### Critical Constraints
- Do not copy class or method signatures into the skill. Reference the source code or library docs directly when exact API is needed.
- Always verify the current module context before applying a recipe.
- Prefer linking to existing docs over duplicating them.

## 2. References

References are chunked by user intent. Each file answers one question: what does the agent need to know to do the right thing in this situation?

### architecture.md
Capture how the library works at runtime and what invariants the agent must respect.

- Execution lifecycle: initialization, steady state, teardown.
- State machine and resource ownership.
- Concurrency and threading model.
- Resource management rules: connection lifetimes, stream closures, memory leaks, cleanup.
- Module boundaries that matter for reasoning.
- Design principles and invariants the library relies on.

Do not list every class, method, or constant. Focus on concepts that are not obvious from reading the public API.

### initialization.md
Capture how to get the library ready for use.

- Required configuration steps and their defaults.
- Environment-specific setup: production, test, development.
- Boot order and dependencies between components.
- Common bootstrap mistakes and how to avoid them.

Keep this file focused on bootstrap and configuration only. Do not use it for arbitrary workflows.

### migration.md
Capture how to move between versions or replace deprecated parts.

- Breaking changes between major versions.
- Deprecation paths and replacement patterns.
- Recommended upgrade sequence and rollback options.

Keep this file focused on version changes only. Do not use it for advanced usage patterns.

### recipes/

Each recipe is a single, concrete library-specific use case. One file per use case.

#### File naming rules
- Use kebab-case.
- Start with a verb that describes the user's goal.
- Good examples: `configure-connection-pool.md`, `publish-message.md`, `handle-retries.md`, `setup-test-environment.md`.
- Bad examples: `integration.md`, `advanced.md`, `core.md`, `utils.md`.

#### Recipe boundaries
- A recipe covers one complete outcome for the user.
- Do not split by difficulty. Basic and advanced variants of the same task belong in one file.
- Do not split by technical layer. A recipe follows the user's task end-to-end, even if it touches initialization, runtime, and cleanup.
- If two use cases share steps, cross-reference them. Do not merge unrelated use cases into one file to avoid duplication.

#### Recipe content
- Why: one sentence explaining why this pattern is used.
- When to use it: context and preconditions.
- Step-by-step instructions.
- Common mistakes and how to avoid them.
- Cross-references to `architecture.md` for runtime constraints.

### troubleshooting.md
Capture how to diagnose and fix failures.

Use a strict table format:

| Error / Symptom | Root Cause | Fix |
|---|---|---|
| ... | ... | ... |

Each entry must be actionable without reading the source code.

## 3. Reference Templates

Use these templates to keep generated references consistent.

### architecture.md template

```markdown
# Architecture

## Lifecycle
1. Initialization
2. Steady state
3. Teardown

## Resource Management
- Connections: ...
- Threads: ...
- Memory: ...

## Concurrency Model
...

## Design Invariants
- ...
```

### initialization.md template

```markdown
# Initialization

## Required configuration
...

## Environment-specific setup
...

## Boot order
...

## Common mistakes
- ...
```

### migration.md template

```markdown
# Migration

## From v1 to v2
...

## Deprecated features
...

## Rollback options
...
```

### recipes/{use-case}.md template

```markdown
# [Verb the Noun]

Why: [one sentence explaining why this pattern is used].

When to use it: [context and preconditions].

Steps:
1. ...
2. ...
3. ...

Common mistakes:
- ...

See also:
- Runtime constraints: `architecture.md#[section]`
- Bootstrap or configuration: `initialization.md#[section]`
- Version changes: `migration.md#[section]`
```

### troubleshooting.md template

```markdown
# Troubleshooting

| Error / Symptom | Root Cause | Fix |
|---|---|---|
| ... | ... | ... |
```

## 4. Generation Workflow

1. Create the target directory `./skills/{skill-name}/` inside the current project. All generated files must live under this path.
2. Analyze the library: identify the runtime model, lifecycle, configuration surface, and real-world tasks the library solves.
3. Discover use cases: look at docs, examples, issue trackers, FAQs, and common integration patterns.
4. Cluster by user goal, not by module or technical layer. Name each use case as `verb-noun`.
5. Write `architecture.md` first.
6. Write `initialization.md` and `migration.md` next — they are required.
7. Write remaining recipe files — one per use case.
8. Write `troubleshooting.md` from known errors and failure modes.
9. Write `SKILL.md` last as a router.
10. Verify against the quality gates below.

## 5. Update Workflow

Use this workflow when the library has changed and the existing skill may need to be updated. Do not regenerate the entire skill from scratch unless explicitly asked.

### Step 1: Read the existing skill
Read the current `SKILL.md` and all files in `references/` to understand what is already covered.

### Step 2: Analyze the library change
Identify what actually changed in the library: new feature, behavior change, bug fix, performance improvement, deprecation, breaking change, new error mode.

### Step 3: Decide if the skill needs an update
Use the decision matrix below.

| Type of change | Does the skill need an update? | Which file to update |
|---|---|---|
| Internal refactor with no public behavior change | No | None |
| New internal class or method, no usage impact | No | None |
| New configuration option or bootstrap requirement | Yes | `initialization.md` |
| New runtime constraint, lifecycle step, or resource rule | Yes | `architecture.md` |
| New public API that enables a common use case | Yes | Add `recipes/{verb-noun}.md` |
| New public API extending an existing use case | Yes | Update existing `recipes/{use-case}.md` |
| Existing use case changed or replaced | Yes | Update existing `recipes/{use-case}.md` |
| Use case no longer relevant | Yes | Remove `recipes/{use-case}.md` and document the removal in `migration.md` |
| Breaking change or deprecation | Yes | `migration.md` |
| New error, warning, or failure mode | Yes | `troubleshooting.md` |
| New routing domain or reading order needed | Yes | `SKILL.md` |

### Step 4: Apply minimal updates
Update only the files affected by the change. Preserve existing structure and wording where possible.

### Step 5: Verify
Run the quality gates. Ensure no unrelated files were changed.

## 6. Quality Gates

Before finishing, confirm:
- [ ] The skill is located at `./skills/{skill-name}/` inside the current project.
- [ ] SKILL.md has no implementation details.
- [ ] `initialization.md` and `migration.md` exist at the `references/` level.
- [ ] No class or method signatures are duplicated from the codebase.
- [ ] Every file in `recipes/` represents exactly one user use case.
- [ ] Recipe file names follow the `verb-noun` convention.
- [ ] There are no catch-all files like `integration.md`, `advanced.md`, `utils.md`, or `recipes.md`.
- [ ] `initialization.md` contains only bootstrap and configuration.
- [ ] `migration.md` contains only version upgrades and deprecations.
- [ ] Every recipe explains why the pattern is used.
- [ ] Troubleshooting entries are actionable without reading source code.
- [ ] Cross-references between references are explicit.
- [ ] The skill would still be useful if the library gains 30% more API surface.

For updates, additionally confirm:
- [ ] Only affected files were changed.
- [ ] Unrelated references were not rewritten.
- [ ] Removed or obsolete recipes are handled explicitly.

## 7. Anti-Patterns

Avoid:
- Copying source code or API docs into the skill.
- Putting `initialization.md` or `migration.md` inside `recipes/`.
- Creating catch-all files such as `integration.md`, `advanced-patterns.md`, or `utils.md`.
- Using `initialization.md` or `migration.md` as dumping grounds for unrelated workflows.
- Dumping multiple unrelated use cases into one recipe file.
- Naming recipe files after modules, classes, or layers instead of user actions.
- Adding `examples/`, `tests/`, or `fixtures/` folders — recipes already cover this.
- Treating the skill as a replacement for the library documentation.
- Regenerating the entire skill for every library change.
