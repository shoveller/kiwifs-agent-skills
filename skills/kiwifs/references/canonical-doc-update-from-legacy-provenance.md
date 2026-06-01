# Canonical doc updates from legacy/provenance notes

Use this reference when the user asks to update an active wiki page with an implementation pattern they remember but cannot locate.

## Pattern

1. Read the target page first so the patch lands in the right section and preserves frontmatter/status.
2. Search the wiki with multiple keyword shapes:
   - exact library/function names, e.g. `lodash-es memoize`, `createDb D1Database`, `D1 client factory`;
   - conceptual terms, e.g. `singleton`, `binding`, `Drizzle client`;
   - semantic search for the desired behavior.
3. Include older canonical pages, `00 Inbox` provenance notes, and `99 System/Archives` results in the search space. Do not stop at current-category results when the user says the pattern existed elsewhere.
4. Read/section the strongest source pages. Distinguish:
   - direct implementation pattern to reuse;
   - adjacent pattern that proves a project convention;
   - raw provenance note that should be summarized, not copied wholesale.
5. Patch the active target page with the smallest durable update.
6. Add `관련 문서` links to the source pages when they are useful future provenance.
7. Lint and health-check the changed page.

## Example distilled from the data-service-layer docs

User wanted the `D1 client factory` in `03.Drizzle schema-query 레이어` to use `lodash-es` `memoize` as a singleton.

Useful provenance:

- `30 Wiki/database/Cloudflare DB 레이어 - Drizzle 컴포넌트 추가.md`
  - contained the older `createDb(bindingDb: D1Database)` factory and optional Sentry instrumentation pattern.
- `30 Wiki/database/Cloudflare DB 레이어 - Better Auth 인스턴스·테이블.md`
  - contained the `lodash-es` `memoize` singleton factory convention.

Resulting canonical pattern:

```ts
import { drizzle } from "drizzle-orm/d1";
import { memoize } from "lodash-es";
import * as schema from "./schema";

export type AppDb = ReturnType<typeof drizzle<typeof schema>>;

type GetDb = (d1: D1Database) => AppDb;

export const getDb: GetDb = memoize(
  (d1: D1Database) => {
    return drizzle(d1, { schema });
  },
  (d1) => d1,
);
```

With Sentry instrumentation:

```ts
export const getDb: GetDb = memoize(
  (d1: D1Database) => {
    const instrumented = Sentry.instrumentD1WithSentry(d1);

    return drizzle(instrumented, { schema });
  },
  (d1) => d1,
);
```

## Pitfalls

- Do not invent the remembered pattern from scratch if the user says it exists in the wiki; search provenance first.
- Do not copy large raw notes into the active canonical page. Extract the implementation rule and link the source.
- Do not turn one project’s exact package aliases into a universal rule. Preserve the class-level pattern and adapt names to the target doc.
