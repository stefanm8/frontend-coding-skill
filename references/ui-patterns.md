# UI patterns

Read this reference when changing layout, visual styling, navigation, responsive behavior, tables, or accessible interaction. Preserve the product's existing visual language and choose patterns that serve its actual workflows.

## Layout and hierarchy

- Keep the active context visible so users can identify what they are viewing or changing. Separate global navigation from navigation within the current entity.
- Make the primary work surface and next action easy to find. Put secondary details in an adjacent pane, drawer, tab, or disclosure when that reduces interruption. Do not wrap every section in a card by default.
- Reuse page headers, filter toolbars, status indicators, and empty states when their semantics and behavior match. Keep feature-specific workflow decisions in the feature.
- When comparing original and edited values, keep both easy to inspect. Expose secondary detail on demand while preserving the selected item and return path.
- Define scroll ownership: one primary page scroll container, or deliberate independent panes when the workflow requires them. Use flexible sizing, `min-width: 0`, and `min-height: 0` where needed; avoid accumulating viewport-height offsets and overflow overrides to repair an unclear layout.

## Theme and styling

- Use a central semantic token source for colors, typography, spacing, borders, radii, and surfaces. Use the library theme for library components and a consistent token bridge for custom CSS.
- Give domain-specific colors semantic names and pair them with text or icons. A status must not be understandable only through color.
- Keep numeric token units correct when exposing CSS variables: lengths may need units even when a library accepts numbers. Keep theme ownership and variable scope clear, including portals used by dialogs and menus.
- Prefer component-scoped styles and supported library customization APIs. Avoid broad overrides of library internals or repeated hardcoded values across features.
- Check all supported themes. A dark preview background alone does not verify a dark component theme.

## Responsive behavior

Decide what changes when space becomes constrained, rather than merely shrinking the desktop view:

| Surface | Adaptation to consider |
| --- | --- |
| Persistent navigation | Drawer or compact navigation using the same items and selection logic |
| Secondary inspector | Drawer, tab, or stacked section that preserves access to its controls |
| Filter/action toolbar | Wrap or stack controls while keeping the primary action reachable |
| Dense table | Intentional horizontal scrolling, prioritized columns, or an appropriate alternate view |
| Comparison view | Switchable panes or stacking that preserves the selected content and draft |

Use shared breakpoint definitions or the project's existing responsive primitives. Keep CSS and JavaScript decisions aligned. Use JavaScript for behavioral changes; use CSS for layout when possible.

Check long labels, translated text, zoom, popup clipping, and mobile viewport/keyboard behavior when relevant. Test scrolling and action reachability in the rendered UI. Do not hide essential functionality solely to make a narrow screenshot fit.

## Navigation and collections

- Put meaningful, shareable view state in the URL. Validate enum values, IDs, dates, and page bounds; use stable defaults for absent or invalid parameters.
- Derive committed view state from the current URL so browser Back/Forward works. Temporary search input can remain local while it is being edited or debounced.
- Update owned parameters without deleting unrelated ones. Use history replacement for transient/default normalization where appropriate, and history entries for meaningful navigation.
- Preserve filters and pagination across list/detail navigation. Reset pagination when a changed filter invalidates the current page, not during rendering or on every background refresh.
- Distinguish an empty collection from no search results and from a failed request. Provide a relevant next action: create/upload, clear filters, or retry.
- Use stable entity IDs for rows, cards, selections, and drafts. Avoid array indices or serialized mutable content as identity for editable/reorderable items.
- Keep row and bulk action targets distinct. Capture affected IDs when opening confirmation and use the same target for its label, count, and submission; unrelated selection changes must not silently retarget the action. Handle partial failures without implying all items succeeded.
- In schema-driven tables, keep schema normalization and shared types separate from rendering. Handle missing values, nulls, nested values, and unsupported types deliberately. Derive fields from the schema or deliberate normalization across rows. Disclose truncation and keep all values inspectable before users act on the data. Keep loading and error feedback as local as the operation allows, such as one row or cell.

## Rich content

Treat imported, stored, and generated HTML as untrusted at HTML rendering boundaries. Prefer text rendering; sanitize required HTML through a maintained, shared policy that constrains supported elements, attributes, and URL/embed protocols. Parsing or passing through an editor is not evidence of sanitization. See [OWASP HTML sanitization guidance](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html#html-sanitization).

## Accessible interaction

- Use buttons for actions and links for navigation. Clickable cards or rows must offer an equivalent semantic keyboard interaction without conflicting nested controls.
- Give inputs persistent labels and icon-only controls accessible names. Preserve visible focus and connect field errors to their inputs.
- Prefer the established library's dialogs, drawers, and menus. Verify initial focus, keyboard operation, Escape dismissal where appropriate, and focus restoration. A styled overlay does not supply dialog behavior.
- Keep destructive actions distinct from the primary workflow and make their scope clear. Use confirmation or undo according to the consequence; do not add confirmations to routine reversible actions indiscriminately.
- Announce important asynchronous status changes without announcing every streamed token. Respect reduced-motion preferences for nonessential animation.
- Apply the application's locale and timezone policy consistently across text, dates, and library controls.

For dialog semantics and keyboard behavior, consult the [WAI modal dialog pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/).
