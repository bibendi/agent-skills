# Library Skill Writer

A skill for writing skills.

When an agent needs to work with a large library — a Ruby gem, an npm package, a framework — it cannot memorize the whole codebase. It needs a compact, practical map of how the library behaves, how to start using it, and what typically goes wrong. This skill generates exactly that map.

## Why this exists

Large libraries have thousands of classes and methods, but most user questions fall into a small set of patterns:

- How do I set this up?
- How do I upgrade from version X to Y?
- How do I do this one specific thing?
- Why is it failing with this error?

A good skill does not try to replace the library documentation. It captures the ideas that are hard to infer from the docs alone: the runtime model, the boot order, the resources you must clean up, the migration traps, the recipes for common outcomes, and the fixes for common failures.

## What it produces

Given a library, this skill creates a directory like this:

```text
{skill-name}/
├── SKILL.md
└── references/
    ├── architecture.md
    ├── initialization.md
    ├── migration.md
    ├── recipes/
    │   ├── configure-connection-pool.md
    │   ├── publish-message.md
    │   └── ...
    └── troubleshooting.md
```

`SKILL.md` is the front door. It tells the agent which reference to read for which kind of question. The references are split by intent, not by module name:

- **architecture.md** — how the library runs, what it owns, what you must not forget to close or release.
- **initialization.md** — how to get from zero to a working setup in different environments.
- **migration.md** — what breaks between versions and how to move forward safely.
- **recipes/** — one file per concrete task, named after what the user is trying to do.
- **troubleshooting.md** — errors, symptoms, causes, and fixes you can act on.

## How it thinks

The skill is opinionated about what belongs where. It does not copy API signatures. It does not list every class. It clusters knowledge around user goals, because that is how agents actually get asked to help.

If you are generating a skill, the workflow is:

1. Understand the library's runtime shape first.
2. Find the real tasks people perform with it.
3. Write one recipe per task.
4. Add the glue files — initialization, migration, troubleshooting — only as needed.
5. Write `SKILL.md` last, as a router.

If you are updating a skill, the workflow is smaller: read what exists, figure out what changed in the library, update only the files that need it, and leave the rest alone.

## Who should read what

- **If you want to use this skill** — read `SKILL.md`. It contains the full instructions, templates, and quality gates.
- **If you are curious about the philosophy** — you are already reading it.
- **If you want to change the workflow** — edit `SKILL.md`, then update this README so the two stay in sync.
