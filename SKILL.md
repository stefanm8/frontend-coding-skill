---
name: frontend-coding
description: Frontend architecture, UI behavior, and verification standards. Use when writing, reviewing, or refactoring frontend code.
---
# Frontend Coding Guidelines

Build maintainable interfaces whose navigation, editing, and asynchronous behavior remain predictable. Apply these defaults within the application's established rules and structure.

## Discover the project

Before changing code, inspect the relevant routes, root providers and layouts, session/permission ownership, bootstrap prerequisites, components, theme, service hooks, installed dependencies, and verification commands. Follow an existing feature through its UI and data flow before introducing a new pattern.

Record useful project conventions and exceptions in `.local/frontend-coding/SKILL.md` when they would help subsequent work. Update an existing record rather than duplicating it. Include actual paths, commands, state ownership, and relevant exceptions; do not copy this entire skill or turn assumptions into project rules.

## Architecture and dependencies

Centralize application-wide policy in shared root providers and one `AppLayout` that serves signed-out, restoring, and signed-in sessions. Keep `AppLayout` outside authentication guards; apply `AuthLayout` or an equivalent guard only to routes that require a session. Prefer shallow composition, adding feature layouts only when they provide shared UI or a necessary scope. Pages consume these contracts instead of repeating provider setup, shell markup, or access policy. Preserve equivalent boundaries and names in established applications.

Use feature-oriented organization with clear responsibilities. Preserve established top-level pages or layouts instead of reorganizing unrelated code. A new feature can use:

```text
features/<feature>/
  layouts/                  Feature layout and navigation
  pages/                    Routed workflow composition
  components/               Feature UI
  services/<resource>/
    api.ts                  Transport and API contracts
    hooks.ts                Queries, mutations, and cache coordination
    types.ts                Shared resource types, when needed
    keys.ts                 Query keys, when useful across callers
    utils.ts                Pure normalization or transformation, when needed
```

Shared layouts, components, services, and utilities belong at the application's shared level only when they have shared consumers or a clear shared responsibility. Do not create empty layers or generic wrappers for anticipated reuse.

- Pages compose workflows; components render and handle local interaction; hooks coordinate state and effects; API modules handle transport. Keep transformations pure and outside rendering when they have independent meaning.
- Services must not import pages or UI components. Pass query inputs from callers; place shared types and constants below both consumers. Keep transport independent of routing, notifications, and React.
- Prefer functions, composition, and simple modules. Do not introduce app-defined classes or abstraction layers without an established project requirement. Give necessary state an explicit owner and lifecycle.
- For new projects, prefer React with TypeScript, TanStack Query, Ant Design, React Router, Vite, and Vitest. Add Ant Design X when its conversational components serve the feature. Preserve established stacks in existing applications.
- Inspect installed versions and official documentation before using unfamiliar APIs. Check compatibility before adding or upgrading dependencies; ordinary feature work does not require unrelated upgrades.

## State and interaction contracts

Choose the owner before adding state:

| State | Default owner |
| --- | --- |
| Fetched server data | TanStack Query |
| Shareable filters, sorting, pagination, selected entities or views | URL, with validated defaults |
| Temporary interaction such as an open popover | Closest relevant component |
| Unsaved edits | Explicit draft scoped to the edited entity |
| Coordinated workflow transitions | Feature hook or reducer when transitions justify it |
| Application-wide context | A provider with an explicit scope and reset lifecycle |

Avoid redundant copies of server or derived data. An editing draft is intentional: define how it is initialized, saved, discarded, and reconciled with refreshed data. Keep render and memoized derivations free of writes, requests, and state setters.

Preserve navigation context and user input. Distinguish loading, refreshing, empty results, errors, and mutation progress. Keep usable content visible during refresh and communicate failures where the user can recover.

Read only the supporting guidance relevant to the change:

- For root providers, auth/app layouts, session bootstrap, authorization, or dependency choices, read [Application foundation](references/app-foundation.md).
- For visual layout, theme, navigation, responsive behavior, tables, or accessibility, read [UI patterns](references/ui-patterns.md).
- For queries, mutations, forms, autosave, uploads, polling, or streaming, read [Async work and editing](references/async-and-editing.md).
- When choosing tests, adding stories, or validating an implementation, read [Verification](references/verification.md).

## Performance and delivery

Split routes and expensive optional modules where useful. Use actual dynamic imports and appropriate loading/error boundaries; wrapping an eagerly imported component in `Suspense` does not split its code. Check the resulting build when code splitting is part of the change. Keep the essential shell readily available.

Use pagination or virtualization for large collections when their size and interaction justify it. Optimize measured rendering or loading problems; memoization is not a substitute for correct ownership and stable identities.

The frontend must compile to static assets and support serving from a reverse proxy such as nginx, unless the application explicitly requires another architecture. Verify direct entry and refresh on nested routes. SPA fallback must not turn missing assets or API errors into successful HTML responses.

## Completion

For behavioral changes, define the observable outcome and verify the meaningful success and failure paths. Use browser tests for critical workflows. For material visual changes, inspect the rendered interface at relevant viewport sizes and states. Run the project's applicable build, type, lint, and focused test checks; report what was actually verified and any material limits. Scale verification to the change rather than adding tests that mirror implementation details.
