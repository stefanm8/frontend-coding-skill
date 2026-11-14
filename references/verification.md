# Verification

Read this reference when choosing tests, adding stories, or validating frontend work. Verify observable behavior and meaningful invariants. Scale the checks to what changed; do not run every scenario for a small edit.

## Choose evidence for the change

| Change | Useful verification |
| --- | --- |
| Pure transformation or schema normalization | Focused examples covering meaningful boundary cases |
| Query/mutation behavior | Cache identity, invalidation, failure handling, and affected views |
| Interactive component | Real component with controlled external dependencies and user interaction |
| Material layout/theme change | Rendered inspection at relevant viewport sizes, themes, and states |
| Critical multi-step workflow | Browser test through the actual navigation and save/recovery behavior |
| Route loading or static hosting | Production build, chunk inspection where relevant, direct nested-route entry and refresh |

For a behavior change, define the expected outcome before implementation and write the focused test first when it clarifies or protects the work. Do not add tests that merely restate markup or check implementation details. Reversible cosmetic edits may need visual inspection without a new automated test.

Run the project's applicable build, typecheck, lint, and focused test commands. Existing Cypress, Playwright, Storybook, or other setups should be reused rather than replaced to match a preferred stack. A package dependency or configuration file is evidence of setup, not evidence that checks passed.

## Components and stories

- Render the production component. Mock external boundaries such as network responses, clock, and authentication where needed; do not replace the component under test with a simplified copy.
- Use the real providers and theme configuration where practical. Isolate query caches and mutable state between tests/stories. Ensure required CSS variables and locale data are present.
- Create stories for materially distinct states: loading, empty, populated, error, disabled, or long content as relevant. Use representative fixtures without real personal data.
- Exercise controls through accessible names and realistic input. Assert the resulting URL, visible results, persisted values, or recovery state—not only that an input's DOM value changed.
- Keep the behavior being tested intact. A mocked search control that drops its `onChange` cannot test application search, even if typing changes the mock input's value.
- Critical workflow checks must fail when their required output is missing. Where helpers can silently accept missing/error results, use a controlled fixture to verify they fail instead of accepting a generic shell element or merely logging success.
- For accessibility checks, distinguish reporting violations from failing the test. Choose enforcement deliberately for the affected components and inspect keyboard behavior as well; an automated scan alone is insufficient.
- Avoid timing-based assertions and blanket dependency suppression. Wait for the expected request, UI state, or completion signal instead of fixed sleeps.

## High-value workflow scenarios

Use the relevant scenarios below as acceptance cases, regression tests, or a manual review guide. When evaluating this skill itself, use a representative implementation task and these outcomes; a frontmatter validator cannot establish behavioral quality.

1. **Filtered list → detail → Back.** Search, sort, and page through results; open an item and return. The list context survives. Refresh and browser Back/Forward restore the expected view. Invalid parameters receive safe defaults without removing unrelated parameters.
2. **Dirty editor during refetch.** Edit without saving, then refresh server data. The draft remains intact. A failed save preserves input and offers recovery. Switching entities cannot apply the previous entity's late save response to the new editor.
3. **Submit while autosave is pending.** Submit the current draft immediately after editing. The submission includes the latest edit, or it is blocked with an actionable save error. Also exercise out-of-order save responses when concurrent saves are possible.
4. **Switch records during asynchronous work.** Start an operation for one record, switch to another, and deliver a late result or event from the old operation. It cannot alter the new record's state. Verify cleanup and retry behavior; for streaming views, also verify that manual scrolling is respected.
5. **Keyboard use on a narrow screen.** Open navigation or a dialog, reach the relevant action, close it, and continue. Focus remains visible, modal focus stays inside, dismissal restores focus, and essential actions remain reachable without clipped content.
6. **Query identity and scope.** Change one response-affecting input, such as a filter, limit, sort order, resource ID, or account. The correct result is fetched or read from its own cache. Prefetch and active fetch agree on key construction; persisted state respects the same scope. Restore account A's cache under account B's verified session and deliver late A responses, including session changes in another tab; no A data may render or be reinserted.
7. **Session bootstrap and recovery.** Enter public pages, a protected deep link, and an auth callback with delayed identity/configuration. Exercise signed-out, denied, expired, offline, and server-failure responses. The shared `AppLayout` remains usable for public pages without private account or resource data; auth guards affect only protected branches. Protected queries/loaders wait for prerequisites; failures offer bounded recovery, and sign-in restores the validated path, query, and hash. Failed setup reads must emit no create request. Within the running client application, session changes and navigation retain the shared shell and provider/client lifetimes while resetting session-scoped state as needed.

## Visual and delivery checks

- Inspect the rendered change, not just the JSX or CSS. Check the primary workflow at desktop and a relevant narrow width; add intermediate breakpoints when layout changes there.
- Inspect affected loading/error states, long text, supported themes, popup placement, focus, and scroll behavior. For editors and split panes, check that controls remain reachable while content grows.
- When lazy loading is part of the task, inspect dynamic imports and production output. `Suspense` around an eager import does not establish code splitting. See [React lazy loading](https://react.dev/reference/react/lazy).
- For static deployment changes, verify nested-route refresh, asset URLs and MIME types, missing-asset responses, and API routing against the intended serving configuration. An HTML SPA fallback is not a valid JavaScript, JSON, or API response.
- Report checks actually run, their outcomes, and material limits such as unavailable authentication or a backend. Separate source inspection, mocked interaction tests, and live browser verification in the report. Do not claim an authenticated workflow or visual state was tested when only its source was reviewed.
