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

For a scoped collection, use the same normalized inputs for the key and the request. This example uses TanStack Query v5.

```ts
import { useQuery } from '@tanstack/react-query';
import { fetchItems } from './api';

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

// The caller supplies the active account and committed list filters.
export function useItems(accountId: string, params: ListParams) {
  return useQuery({
    queryKey: itemKeys.list(accountId, params),
    queryFn: ({ signal }) => fetchItems(accountId, params, signal),
  });
}
```

When the account or filters change, the hook observes the new key and fetches according to its freshness policy. Avoid a second fetch in `useEffect` or mirroring the result into component state. Keep an editing draft when the user needs to change a value. Use `refetch()` for a retry or user-requested refresh of the current query. See [TanStack Query keys](https://tanstack.com/query/v5/docs/framework/react/guides/query-keys).

For adjacent pages, prefetch with the same key factory. If keeping rows visible during a page turn helps, use `placeholderData` only when the account, filters, sort, and other scope inputs still match. Use `isPlaceholderData` to mark the transition and guard actions that require the new page. See [Paginated queries](https://tanstack.com/query/v5/docs/framework/react/guides/paginated-queries).

Choose how the view learns about external changes. For ordinary reads, configure `staleTime` for the resource and use stale-query refetch on mount, focus, or reconnect as appropriate. To discover work started elsewhere while the page stays open, subscribe to collection events or poll the visible collection even when it has no active jobs. See [TanStack Query defaults](https://tanstack.com/query/v5/docs/framework/react/guides/important-defaults).

## Feedback and mutations

Resolve requests only when HTTP status and the expected response shape satisfy the endpoint contract. Preserve status and useful structured error details, including non-JSON failures. A failed or unknown read is not an empty result. Automatic setup may create resources only after authoritative successful absence, with idempotent behavior and deduplicated in-flight work; await creation before dependent reads and expose recovery.

| Situation | What the user sees |
| --- | --- |
| Query waiting for required input | Prompt for the missing selection. In v5 an idle, disabled query can be pending before any request starts. |
| First request in flight | A skeleton or loading label in the region being fetched. In v5, `isLoading` means pending and fetching. |
| Background fetch with usable data | Keep the list or detail mounted. Show a refreshing indicator only for a refresh the user requested, tracked around the `refetch()` call, because `isRefetching` is also true during polls and focus or reconnect refetches. Keep routine polls silent. |
| Empty success | Explain the empty state and offer a relevant next action |
| First fetch fails without data (`isLoadingError`) | Show the error and retry action in the affected region. |
| Refresh fails with usable data (`isRefetchError`) | Keep the rows, focused control, and draft mounted. Place a freshness warning and retry beside the content. |
| Mutation pending | Show “Saving…” on the affected form or row and keep unrelated actions usable. Guard duplicate submissions in the submit handler. When the focused control must remain reachable, keep it mounted and expose `aria-disabled` while guarded. |
| Mutation failure | Keep the draft and show the error beside the action with a safe recovery path. |
| Mutation success | Show the confirmed change in place. A quiet “Saved” beside the editor helps when the result would otherwise be invisible. |
| Multi-item operation | Report item-level outcomes and remaining work |

For independently editable rows, give each row its own `useMutation` observer, or select concurrent mutations by key and entity ID with `useMutationState`. One shared observer follows its latest call and can otherwise move “Saving…” to the wrong row.

Do not synthesize completion percentages from elapsed time. Use indeterminate feedback when progress is unknown; use determinate progress only for measurable work. Reserve full-screen blocking for operations that genuinely make the whole interface unusable.

Apply an authoritative save result to the data the user actually sees. Use `setQueryData` for a detail entry and update matching visible list rows immutably when their membership and ordering are known. When a filtered or paginated list needs server reconciliation, the editor can display the acknowledged result until the list catches up. Reconcile only the submitted draft revision so a late success preserves newer typing.

Invalidate affected lists, counts, or details with `queryClient.invalidateQueries`. Return or await the promise in the `useMutation` hook's `onSuccess` when the workflow waits for that refresh. By default the promise can resolve after a failed refetch. Read the query's error state, keep the confirmed save visible, and offer to retry the read. Await required server work before navigating or declaring completion. Use the API's acknowledgement or job status as the completion signal. See [Invalidations from mutations](https://tanstack.com/query/v5/docs/framework/react/guides/invalidations-from-mutations).

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
- When a mutation returns a job ID, show the accepted job inline and observe its status by ID. Apply scoped server events (SSE or WebSocket) to the cache or invalidate the affected key. When polling a known job, use `refetchInterval` until its last known server state is terminal, then return `false`. A missing first result or transient read error needs bounded recovery even if status is still unknown. Checking status again must never resubmit the job.
- Preserve partial output and a useful retry path after failure. Validate results against the operation's response contract; distinguish valid output, empty results, and malformed output.
- When streaming into a scrollable view, follow new content only while the user is following it. Stop automatic scrolling when they read earlier content; provide a clear way to return to the latest output. Preserve the selected context across updates.
- Keep transport, workflow transitions, and rendering separate enough to test independently. Avoid hidden global events or mutable singletons when scoped props, hooks, or context can express ownership.
