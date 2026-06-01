# GitHub Wiki 하위 페이지를 KiwiFS에 저장하는 패턴

Use when the user asks to scrape or archive all pages under a GitHub Wiki such as `https://github.com/<owner>/<repo>/wiki` into KiwiFS.

## Workflow

1. Discover the complete page set from the Git-backed wiki repository, not from the rendered Home page alone:
   - Wiki git URL shape: `https://github.com/<owner>/<repo>.wiki.git`
   - Example: `git clone --depth=1 https://github.com/tmux/tmux.wiki.git /tmp/tmux.wiki`
   - Count/list `*.md` files to verify expected coverage.
2. Search KiwiFS first for existing pages to avoid duplicates.
3. Choose a stable folder under the nearest category, typically:
   - `30 Wiki/<category>/<Project> GitHub Wiki/`
4. For each page, use KiwiFS `clip` against the rendered GitHub Wiki URL so frontmatter includes source URL and the page is committed through KiwiFS:
   - Home: `https://github.com/<owner>/<repo>/wiki`
   - Subpage: `https://github.com/<owner>/<repo>/wiki/<Page-Slug>`
5. Create a `README.md` index in the same folder with:
   - original wiki URL
   - wiki git repo URL
   - list of stored pages as wiki links plus source URLs
   - collection note: number of Markdown files discovered and pages stored
6. Lint the index and every stored page; then tree-check the folder and report exact page URL(s).

## Translation variant

If the user asks to make the archived GitHub Wiki readable in Korean, do not only clip rendered pages or create a link index. Create a separate Korean directory such as `30 Wiki/<category>/<Project> GitHub Wiki 한국어/` containing:

- translated versions of every discovered `*.md` page;
- a Korean `README.md` index linking to all translated pages;
- frontmatter with `doc_type: translated-external-wiki`, `tags: [tmux/github-wiki/translated/ko]`-style metadata, `source_url`, and `source_repo`;
- a clear note if machine translation was used and code/option names were preserved.

For machine-translation pipelines, protect code fences, inline code, URLs, HTML tags/images, and Markdown link destinations before translating. After translation, normalize glued fence markers (for example prose followed by ``` or `~~~~`) so fences are on their own lines, then run KiwiFS lint on every translated page.

## Pitfalls

- Do not infer “all pages” from the GitHub rendered sidebar alone; clone the `.wiki.git` repository to find every Markdown page.
- KiwiFS `clip` may produce cleaner KiwiFS metadata than manually writing raw Markdown, but inspect at least one clipped page to ensure body content was captured, not just GitHub chrome.
- If the user explicitly asks for translated/readable pages, a source-link archive is insufficient; store full translated page bodies in one directory and verify the index points to those local translated pages.
- Avoid documenting transient local import/temporary-file failures as durable rules. If `import` cannot see a local temp file, prefer the `clip`/`write` path for this archive task rather than declaring import unsupported.
- Preserve source URLs; do not rewrite external GitHub links into internal wiki links unless intentionally curating a derivative guide.

## Reporting format

For Korean users, report concisely:

- `Yes. 저장했습니다.`
- 저장 위치
- 인덱스 absolute URL
- discovered Markdown page count
- stored page count
- lint result
