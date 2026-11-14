# Async work and editing

Read this reference when changing data fetching, mutations, forms, autosave, uploads, polling, or streaming. Define ownership, identity, and terminal states before wiring UI feedback.

For session bootstrap, root providers, and route access, also read [Application foundation](app-foundation.md).

## Queries and cache identity

- Centralize keys in the resource's service module. Use the same factory for fetching, prefetching, cache updates, and invalidation. Keep list/detail keys distinct and key inputs serializable.
- Include every input that changes the returned data: account scope, resource ID, filters, sorting, pagination, date range, timezone, and options such as a result limit. Do not put credentials in keys.
- Normalize inputs once and use those values for both the key and request. Prefer a named parameters object when positional arrays would make omissions or ordering mistakes easy.
- Enable dependent queries only when required identity and inputs are ready. Pass cancellation signals through supported transport APIs. Reset or isolate cached data when its account or resource scope changes.
- Set freshness according to the resource. Avoid one long global lifetime for both reference data and rapidly changing workflow state.
- Keep prior results during a refresh only when they still belong to the current scope and the UI makes their status clear. Do not carry another account's or entity's data into a new context.
- Treat browser persistence as a deliberate feature. Persist only suitable data, scope it to the account, and define expiry, versioning, and logout cleanup. A provider name does not establish those guarantees.

Example key shape for a scoped collection; adapt the resource prefix and scope inputs to the API:

```ts
type ListParams = {
  page: number;
  pageSize: number;
  search: string;
  sortBy: 'name' | 'updatedAt';
  order: 'asc' | 'desc';
};

export const itemKeys = {
  lists: (accountId: string) =>
    ['items', accountId, 'list'] as const,
  list: (accountId: string, params: ListParams) =>
    [...itemKeys.lists(accountId), params] as const,
};
```

Use the same `list` factory and normalized parameters for an adjacent-page prefetch. Read the installed version's APIs for invalidation and placeholder data; do not copy signatures across major versions. See [TanStack Query keys](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys).

## Feedback and mutations

Resolve requests only when HTTP status and the expected response shape satisfy the endpoint contract. Preserve status and useful structured error details, including non-JSON failures. A failed or unknown read is not an empty result. Automatic setup may create resources only after authoritative successful absence, with idempotent behavior and deduplicated in-flight work; await creation before dependent reads and expose recovery.

| Situation | Expected behavior |
| --- | --- |
| Initial fetch without usable data | Local skeleton or loading state with stable layout |
| Background refresh | Keep usable content; show unobtrusive refresh feedback if useful |
| Empty success | Explain the empty state and offer a relevant next action |
| Fetch failure | Show an error and recovery action; preserve usable stale data when appropriate |
| Mutation pending | Mark the affected action/item busy and prevent duplicate submission |
| Mutation failure | Keep the user's input, explain the failure, and allow a safe retry |
| Multi-item operation | Report item-level outcomes and remaining work |

Do not synthesize completion percentages from elapsed time. Use indeterminate feedback when progress is unknown; use determinate progress only for measurable work. Reserve full-screen blocking for operations that genuinely make the whole interface unusable.

After a successful mutation, update from the authoritative result or invalidate the affected keys. Await dependent work before navigating or reporting the whole workflow complete. Do not use arbitrary delays to imply that a server transaction has finished; follow the API's completion contract or track its job status.

Use optimistic updates when the operation is reversible and reconciliation is understood. Cancel conflicting fetches where needed, snapshot the affected data, roll back on failure, and reconcile on completion. When optimistic operations overlap, rollback and reconciliation must preserve newer changes; serialize conflicting operations when they cannot be merged safely. Require server confirmation before reporting a consequential operation as complete.

## Forms, drafts, and autosave

- Give a form one owner for values, validation, dirty state, and submission. Avoid overlapping form state systems without a concrete integration need.
- Initialize a draft for an entity deliberately. Background query updates must not silently overwrite dirty input or close editing mode. Reconcile clean drafts automatically; resolve changed server data according to the product's conflict policy when the draft is dirty.
- Preserve values and useful field errors after a failed save. Reset dirty state only after the relevant version has been acknowledged as saved.
- Represent autosave states explicitly: unchanged, dirty, saving, saved, and failed as needed. Keep feedback near the editor; avoid a toast for every keystroke batch.
- Scope debounce timers and requests to the edited entity. Clean up timers on disposal and prevent late responses from overwriting newer edits or another entity's draft. Use serialization, revisions, or another explicit ordering mechanism when saves can overlap.
- Before an action consumes the draft, flush and await pending saves or submit the current draft atomically. Prevent the dependent action when saving fails.
- Define what happens when switching records or leaving with unsaved work: save, retain a recoverable draft, or offer discard/cancel. A `beforeunload` listener alone does not cover client-side navigation or record selection.

## Long-running and streaming workflows

- Use explicit states and transitions when several booleans can represent contradictory conditions. A feature reducer is often sufficient; do not add a state-machine dependency solely for a short workflow.
- Associate events with the account, active entity or view, and operation ID that owns them. Ignore stale events after switching context. Reconcile incoming data by stable identity and revision/order.
- Give each stream, subscription, polling loop, upload, and timer an owner that disposes it on completion, failure, replacement, or unmount. Distinguish stopping observation from cancelling the server operation; expose cancellation only when supported.
- Poll while the server reports nonterminal work. Stop at terminal states and define behavior for errors, disconnects, and stalled jobs. Reconnection or retry must not duplicate the underlying mutation.
- Preserve partial output and a useful retry path after failure. Validate results against the operation's response contract; distinguish valid output, empty results, and malformed output.
- When streaming into a scrollable view, follow new content only while the user is following it. Stop automatic scrolling when they read earlier content; provide a clear way to return to the latest output. Preserve the selected context across updates.
- Keep transport, workflow transitions, and rendering separate enough to test independently. Avoid hidden global events or mutable singletons when scoped props, hooks, or context can express ownership.
