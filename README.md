# Frontend Coding Skill

Reusable guidance for building, reviewing, and refactoring frontend applications with predictable navigation, editing, and asynchronous behavior.

This repository contains one skill and its supporting references. It provides instructions for a coding agent, rather than an application starter or a component library.

## What it covers

- Shared application layouts that support both public and authenticated routes.
- Clear boundaries between components, stateful hooks, and API transport.
- Query identity, cache isolation, mutations, drafts, autosave, and streaming.
- Accessible interaction, responsive layouts, semantic themes, and complete data views.
- Proportionate testing, visual verification, code splitting, and static delivery.

For new React projects, the skill prefers TypeScript, TanStack Query, React Router, Ant Design, Vite, and Vitest. Supporting guidance includes React Testing Library and optional Ant Design X. Existing project architecture, dependencies, and versions take precedence.

## Repository structure

```text
.
├── README.md
├── SKILL.md
└── references/
    ├── app-foundation.md
    ├── async-and-editing.md
    ├── ui-patterns.md
    └── verification.md
```

Start with [SKILL.md](SKILL.md). Load supporting references only when relevant:

| Reference | Use for |
| --- | --- |
| [Application foundation](references/app-foundation.md) | Root providers, shared AppLayout, authentication boundaries, and dependency roles |
| [Async work and editing](references/async-and-editing.md) | Queries, mutations, cache ownership, drafts, saves, and long-running operations |
| [UI patterns](references/ui-patterns.md) | Layout, navigation, styling, responsive behavior, and accessibility |
| [Verification](references/verification.md) | Component tests, workflow scenarios, visual checks, and deployment checks |

## Installation

For a local Codex installation using the default skills directory, clone this repository into a folder named `frontend-coding`:

```sh
mkdir -p ~/.codex/skills
git clone https://github.com/stefanm8/frontend-coding-skill.git ~/.codex/skills/frontend-coding
```

If that directory already exists, update the existing installation instead of cloning over it. If your agent uses a different skills location, place the repository there as `frontend-coding`, keeping `SKILL.md` and `references/` together.

## Usage

Invoke the skill explicitly in a coding request:

```text
Use $frontend-coding to review this feature's state ownership and save behavior.
```

```text
Use $frontend-coding to implement this screen within the existing application architecture.
```

The skill directs the agent to inspect the project first, preserve established conventions, and choose verification appropriate to the change. Its examples use neutral resource terminology and can be adapted to the application's domain.

## Updating a cloned installation

```sh
git -C ~/.codex/skills/frontend-coding pull --ff-only
```

Review any local changes before updating. No application dependencies are installed by this repository.
