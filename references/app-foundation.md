# Application foundation

Read this reference when establishing or changing root layouts, providers, session handling, route access, or dependencies. Centralize shared policy while keeping resource state and workflows within their relevant scope.

## Root composition and ownership

For a new application, shared entry points can live at `src/app/providers.tsx`, `src/router.tsx`, and `src/layouts/{AuthLayout,AppLayout}.tsx`; feature-specific layouts remain under their feature. Preserve equivalent existing paths.

Use a shallow shared root for both public and protected routes; adapt names, provider order, and router APIs to the existing application:

```text
Root providers and router
  AppLayout: shared across session states
    Public pages: sign-in, auth callback, recovery, public content
    AuthLayout / RequireAuth: only for protected routes
      Protected pages
```

`AppLayout` mounts without requiring an authenticated user. It handles public navigation, session restoration/recovery presentation, and signed-in chrome using the session contract. Public content must remain usable without protected account or resource queries. Within the running client application, keep the shared shell mounted across session changes and ordinary navigation; reset only state whose session or resource scope changed.

Here `AuthLayout` or `RequireAuth` is an optional guard around protected routes. Login and callback routes remain reachable while session restoration is unresolved. A visual sign-in layout may use the application's existing naming. Use a shared route outlet and add guards or feature layouts only where needed. The responsibilities below do not each require a separate wrapper. See [React Router layout routes](https://reactrouter.com/start/data/routing#layout-routes).

| Boundary | Responsibility |
| --- | --- |
| Application providers | Compose runtime configuration, stable query client, session integration, theme, locale, and shared feedback infrastructure in dependency order |
| `AppLayout` | Shared navigation, header, responsive shell, primary scroll ownership, and an outlet for public and protected pages; adapt UI to session state without requiring a user |
| `AuthLayout` / auth guard | Consume the session owner; handle required identity/profile readiness, sign-in redirects, expiry and recovery for protected children only |
| Resource scope | Resolve the selected entity and associated permissions using a focused hook/provider; add a layout when routes share context-specific UI |
| Authorization guard | Check the capability needed by its route/action; distinguish checking, allowed, denied, and failed |
| Feature layout/page | Compose the workflow and own its drafts, selection, and operations at the narrowest useful scope |

Compose common providers once at a stable boundary. Providers using query hooks require query context; those using navigation hooks require router context. This is a dependency order, not a universal JSX nesting recipe. Theme and feedback context must cover public screens, errors, and portalled dialogs as needed. With Ant Design, place its `App` context beneath `ConfigProvider` and use context-aware feedback APIs; this library component is distinct from the application's `AppLayout`. See [ConfigProvider](https://ant.design/components/config-provider/) and [Ant Design App](https://ant.design/components/app/).

Keep one stable `QueryClient` for the mounted browser application, with isolated clients for tests and per-request isolation where an established app uses SSR. Do not recreate it on page navigation or ordinary renders. Give session changes an explicit cache/operation reset policy. See [stable query clients](https://tanstack.com/query/latest/docs/eslint/stable-query-client).

Root layouts delegate to focused hooks/services; they should not collect every feature query, draft, or mutation. Keep error and lazy-loading boundaries near the affected route so a feature failure can preserve the shell. Place authorization at the narrowest scope with the required context. Access denial for one resource must not automatically invalidate the entire session.

## Session readiness and recovery

- Identify the established session owner: SDK, server, or proxy. Reuse its credential acquisition and renewal lifecycle; a token copied once into context does not establish refresh behavior. Keep request construction and structured errors in transport, session transitions in the session owner, and redirects/feedback in the UI integration. Consult the installed session integration's version-matched documentation.
- Distinguish restoring, loading required configuration/profile, signed out, ready, denied, and failed. Unknown or failed permission checks do not grant access. Offer bounded retry or reauthentication for the appropriate failure; a network/server error must not become endless loading or an automatic reload loop. Pending or failed private bootstrap gates its protected branch while the public shell remains usable.
- Gate protected queries, loaders, prefetches, and streams on their actual prerequisites. A rendered layout guard alone does not protect data work started outside it, including independently scheduled route loaders. Server authorization remains authoritative.
- Preserve a validated in-app return destination, including query and hash, through sign-in. Complete callbacks before redirecting again. Keep public and recovery routes available without requiring the state they are meant to establish.
- Verify current identity before exposing persisted private data. On logout or account changes, including changes in other tabs, cancel cancellable requests, ignore results from superseded session/operation identities, and isolate/reset owned caches and drafts. Late callbacks must not restore the previous account's data. See [query and operation ownership](async-and-editing.md).

## Dependency roles

These are preferences for new React projects, not a migration or blanket installation request. Existing packages, versions, and architecture take precedence. Check the manifest, lockfile, peer compatibility, and version-matched documentation before adding a dependency.

| Concern | Preferred choice and role |
| --- | --- |
| Rendering and types | React with TypeScript; components and hooks own UI composition and local interaction |
| Server state | `@tanstack/react-query`: queries, mutations, cache coordination, cancellation and invalidation through resource hooks; keep drafts and transient UI state with their own owners. See [TanStack Query](https://tanstack.com/query/latest/docs/framework/react/overview) |
| Navigation | React Router: nested layouts, URL state, route loading/error handling. Preserve an established alternative such as TanStack Router; use one routing system |
| UI and forms | Ant Design and its icon library: shared tokens, controls, dialogs, feedback, and a single form owner. Reuse the existing form solution when present |
| Conversational UI | Add [Ant Design X](https://x.ant.design/docs/react/introduce/) only when its components serve the feature; operation identity, persistence, and retries still need explicit ownership |
| Development/build | [Vite](https://vite.dev/guide/) for the development server and production assets; preserve established tooling |
| Focused tests | [Vitest](https://vitest.dev/guide/) and [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) for production components and observable interactions; reuse existing browser-test tooling for complete journeys |

TanStack Query does not imply adopting the whole TanStack suite. Add table, virtualization, form, chart, editor, or other packages only when the feature needs them and the current stack does not already provide the capability. Avoid competing routers, form owners, or server-data caches. Where route loaders and Query coexist, deliberately share the query/cache contract rather than fetching and storing the same resource independently.
