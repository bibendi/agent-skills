# agent-skills

A public collection of [Vercel Skills](https://github.com/vercel-labs/skills) for software development.

Each skill lives under `skills/<name>/` and contains a `SKILL.md` file that teaches the agent how to handle a specific library, framework, tool, or workflow.

## Installation

Install any skill from this repository using the Skills CLI:

```bash
npx skills add bibendi/agent-skills --skill <name>
```

Replace `<name>` with the skill directory name (for example, `lib-skills-writer`).

## Skills

### Software Development

Skills that help agents work with code, libraries, and frameworks.

| Name | Description |
|------|-------------|
| [lib-skills-writer](./skills/lib-skills-writer) | Teaches the agent how to architect and write multi-file Agent Skills for large libraries (gems, packages, frameworks), and how to keep them up to date. |

### Team Communication

Skills that help agents summarize and share outcomes from engineering meetings.

| Name | Description |
|------|-------------|
| [follow-up](./skills/follow-up) | Creates follow-up emails from technical meetings: summaries, action items, decisions, and risks. |

---

More skills will be added as the collection grows.
