# KiwiFS Mermaid renderer contrast patch

Use this reference when operating or patching the KiwiFS application itself and Mermaid diagrams render with unreadable text (for example white labels on light diagram nodes). This is an application-renderer concern, but the renderer must not erase per-diagram Mermaid configuration.

## Durable lesson

Do not fix Mermaid contrast by globally forcing `text`, `tspan`, `.nodeLabel`, or `foreignObject *` to black with `!important`. That makes the current broken diagram readable but prevents document-level `%%{init: ...}%%` theme/themeVariables/font settings from working.

The safer KiwiFS fix has two layers:

1. Stop the application dark/light mode from becoming an implicit Mermaid theme override. Initialize Mermaid with the neutral/default theme as the app baseline, and let per-diagram Mermaid directives opt into custom themes or variables.
2. Isolate the rendered Mermaid SVG from KiwiFS app CSS. App-level selectors from `.dark`, `.kiwi-prose`, Tailwind/base styles, or other UI CSS can still leak into inline SVG and override Mermaid labels. Render the SVG inside a Shadow DOM host so Mermaid's own inline/style-block output remains authoritative.

## Patch shape

In `ui/src/components/MermaidDiagram.tsx`:

1. Keep `mermaid.initialize({ startOnLoad: false, securityLevel: "strict", theme: "default" })` as the app-level baseline.
2. Store `rendered.bindFunctions` in a ref, because the SVG is no longer inserted directly with `dangerouslySetInnerHTML`.
3. Replace direct inline DOM injection with a Shadow DOM mount:
   - Create/reuse `containerRef.current.attachShadow({ mode: "open" })`.
   - Set `root.innerHTML` to a small local style plus Mermaid's rendered SVG.
   - Include only sizing/layout rules in the Shadow DOM style, for example `svg { display: block; width: 100%; max-width: 100%; height: auto; }`.
   - Bind Mermaid functions against the Shadow root after injection.
4. Do not add global `themeCSS` with `!important` text/color rules.
5. Do not post-process rendered SVG to force text color.
6. Add a comment near render initialization explaining that app color mode must not override per-diagram `%%{init: ...}%%` theme/themeVariables settings.
7. If a single document intentionally needs a hard color/font override, put it in that diagram's `%%{init: ...}%%` block as `themeCSS`; do not bake it into the app renderer. Mermaid flowchart labels may not all honor `themeVariables` alone, so per-diagram `themeCSS` can be the correct authoring-level override.

Representative minimal patch:

```ts
const bindFunctionsRef = useRef<((element: Element) => void) | undefined>(undefined);

// Keep the application color mode from becoming an implicit Mermaid
// theme override. Mermaid's dark theme can emit light text on nodes that
// remain light, and app-level theme changes should not take precedence
// over per-diagram %%{init: ...}%% theme/themeVariables settings.
const mermaid = await ensureInit("default");
const rendered = await mermaid.render(renderIdRef.current, chart);
bindFunctionsRef.current = rendered.bindFunctions;
setSvg(rendered.svg);

useEffect(() => {
  const host = containerRef.current;
  if (!host || !svg) return;

  const root = host.shadowRoot ?? host.attachShadow({ mode: "open" });
  root.innerHTML = `<style>
    :host { display: block; }
    svg { display: block; width: 100%; max-width: 100%; height: auto; }
  </style>${svg}`;
  bindFunctionsRef.current?.(root as unknown as Element);
}, [svg]);
```

Important TypeScript detail: Mermaid's `bindFunctions` type expects a required `Element`, so type the ref as `((element: Element) => void) | undefined`, not `((element?: Element) => void) | undefined`.

## Document-level Mermaid color normalization

When the user asks to change Mermaid label/text color across a wiki subtree, treat it as an authoring-level document edit, not an application renderer patch.

Recommended workflow:

1. Enumerate the target subtree first and limit edits to markdown files in the requested path.
2. Replace only explicit Mermaid text-color directives the user named or that are clearly label-related, for example:
   - `primaryTextColor`, `secondaryTextColor`, `tertiaryTextColor`
   - `textColor`, `nodeTextColor`
   - per-diagram `themeCSS` label rules
   - Mermaid `classDef ... color:` and `style NODE color:` lines
3. Preserve non-text visual semantics such as node fills, strokes, `mainBkg`, and `lineColor` unless the user asks otherwise.
4. For bulk edits, a direct repository/container scan can be more reliable than full-text search immediately after writes, because FTS indexes may lag or return stale snippets. Verify with a direct literal scan of the files for the old color value.
5. Run KiwiFS lint on changed pages. If KiwiFS has an fswatch/git auto-commit process, check whether it already committed the filesystem edits before attempting a manual commit.

## Verification checklist

- Verify the patch applies against the configured KiwiFS source ref.
- Rebuild the KiwiFS frontend/container that serves the patched bundle.
- Restart/recreate the production KiwiFS service that serves the UI.
- Confirm container health and HTTP serving status.
- Inspect the served JS bundle, not just the source patch, for stable strings such as `attachShadow`, `implicit Mermaid`, or `ensureInit("default")`.
- If the host-side published port is not reachable from the current container, verify from inside the `kiwifs` container with `wget http://127.0.0.1:<port>/...`; avoid recording that as a durable environment failure.
- Ask the user to hard-refresh only after the patched bundle is confirmed, because the browser may cache the previous JS bundle.

## Reporting

Report this as an application-level renderer/theme-boundary fix: KiwiFS no longer lets its own dark mode force Mermaid's dark theme over diagrams. Mention that document-specific Mermaid config remains available for diagrams that need explicit colors/fonts.