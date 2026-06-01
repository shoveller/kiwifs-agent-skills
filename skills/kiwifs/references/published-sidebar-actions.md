# Published sidebar/action menu patterns

Use this when extending KiwiFS publish visibility or public-page actions in the UI.

## Durable lessons from the publish visibility patch

- Do not show `Publish` and `Unpublish` together for a single page context menu. Render exactly one branch:
  - unpublished markdown page: `Publish`
  - published markdown page: `Copy public link`, `View published page`, `Unpublish`
- Reuse the same action vocabulary as the top-level `Published` button so users do not have to learn two menus for the same state.
- Published-list items should be action affordances, not just navigation links, when the user's goal is publish-state management. Clicking a published item can open a Popover with the same actions as the `Published` button.
- For published-list actions, fetch `api.publishStatus(path)` lazily when the popover opens/renders if view counts or canonical `public_url` are needed. Fall back to the list item's `public_url` or `/p/<path>` for copy/view actions.
- After `api.unpublish(path)` from any sidebar/published-list menu, trigger the same published-list refresh callback used by the main page publish controls.

## Radix/React pitfall

`PopoverTrigger asChild` needs a real DOM element or a component that forwards refs/props. If wrapping an internal helper component that does not forward props/ref (for example a simple `SidebarPageItem` function), the trigger may not open. Prefer rendering the button directly inside `PopoverTrigger asChild`, or refactor the helper with `forwardRef` and prop spreading.

## Verification checklist

- `npm run typecheck`
- `npm run build`
- If packaged as a downstream patch, regenerate the patch artifact and run `git apply --check` against a clean upstream tag.
- Smoke the generated UI bundle for stable text markers such as `Copy public link`, `View published page`, and the internal action marker used for the sidebar popover.
