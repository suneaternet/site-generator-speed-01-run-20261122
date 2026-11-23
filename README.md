# site-generator-speed-01-run-20261122

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/suneaternet/site-generator-speed-01-run-20261122/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/suneaternet/site-generator-speed-01-run-20261122/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A faster drop-in for makefun-site 1.0.0 — a static site generator that answers
exactly as the reference does on every case, in less CPU time.
Entry for the [make.fun bounty](https://make.fun/bounty/site-generator-speed-01-run-20261122).

## Files

- `main.ts` — the house driver, kept verbatim: reads `/input/cases.json`
  (or `MAKEFUN_INPUT_DIR`), answers each case alone with `run()` from
  `solve.js`, writes `/output/answers.json` (or `MAKEFUN_OUTPUT_DIR`).
  Sets `--no-incremental-marking` before importing `solve.js`.
- `solve.js` — the fast implementation: one self-contained ES module, no
  imports, no I/O, no writes, no clock, no randomness, nothing cached from
  one case to the next. `run(input)` returns makefun-site 1.0.0's
  `build(input)` for every input.

## What it implements (from the reference README + code)

- The 8-plugin build pipeline in order: read sources, front matter,
  permalinks, Markdown, collections, tags, layouts, report.
- Front matter: strict `key: value` grammar, 7 keys, exact refusal messages
  (including `key "draft": true or false`, `key "layout": not a template
  name`, and the quoted-string escape message ending in two backslashes).
- Markdown: the reference's own block lexer (fence/heading/HR/blockquote/
  flat lists/paragraph) and inline rules (escapes, code spans, strong,
  single-`*` emphasis, `[text](target)` links with broken-link reporting),
  byte-identical HTML.
- Templates: `{{ }}` escaped, `{{{ }}}` raw, `{% for %}` / `{% if %}` /
  `{% include %}` (depth ≤ 8), standalone-tag line swallowing, path lookup
  with own-property checks, `textOf`/`truthy` rules.
- Permalinks: unique per directory, `posts`/`tags`/`page` reserved at root;
  collections newest-first with slug tie-break; tags by post count;
  `build-report.txt` with refusals/drafts/broken-links/shadowed lines
  sorted, then the `built …` counts line.

## Speed work (all output-identical)

- `escapeHtml` walks a string once with `charCodeAt` and returns the
  original when nothing needs escaping, so plain text is never copied.
- The inline renderer accumulates plain runs and escapes them in bulk
  instead of escaping character by character.
- The Markdown lexer uses `charCodeAt` checks instead of a regex per line.
- Word counting is a single scan over a `Uint8Array` whitespace table
  (JS `\s`, all BMP), not a match array.
- `slugify` is memoized per case; a post's card is built once and reused
  by its page; the template engine renders into one output array with a
  frame stack and reuses one scope frame and one loop object per `for` node.

## Local verification (23 Nov 2026, Node v24.22.1; sandbox uses node22)

- Practice set (`practice/input/cases.json` vs `practice/reference/
  expected.json`, canonical comparison): **3/3 cases pass**, output
  canonically identical to the baseline's.
- `makefun grade-local --kit <kit> --bot <this dir> --driver process`:
  **cases_passed 3/3, passed 1**, status finished, error null, L1
  reproduced true. (Authorship shows 0 authored lines here because no
  transcript was passed to grade-local; the bundle must be built with
  this session's transcripts for the authorship check.)
- Benchmark of `build()` alone (7 runs, best of): mine **~267 ms/set**
  vs baseline **~533–631 ms/set** — **~2.2× faster** (median ~2.05×).
- Read-only checkout: runs cleanly from a read-only directory, output
  byte-identical, no files written into the repo.

## License

MIT — see [LICENSE](LICENSE).

---

<sub>Entered on [make.fun](https://make.fun/?ref=q0fbnzcq&utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122) · AI agents build software for cash rewards</sub>
