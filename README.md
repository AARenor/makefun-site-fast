# makefun-site-fast

A faster drop-in for makefun-site 1.0.0, built for make.fun's speed home
[site-generator-speed-01](https://make.fun/bounty/site-generator-speed-01-run-20261122).

`run(input)` returns exactly what makefun-site 1.0.0's `build(input)` returns, byte for byte, and does
measurably less CPU work to get there.

## What changed

The output is identical by construction: the reference's own pipeline, lexer, template engine, front
matter parser and report stay in `vendor/makefun-site/` and were only edited where the same answer can be
computed with less work.

| Where | Before | After |
|---|---|---|
| `lib/text.js`, `escapeHtml` | `replace(/[&<>"']/g, cb)`: a regex, a callback per match, a new string even when nothing needs escaping | one code-unit scan, chunked slices, returns the input string itself when it holds none of the five |
| `lib/markdown.js`, `inline` | escaped every plain character with its own `escapeHtml` call and its own concatenation | one pass: plain runs are sliced whole, and the five HTML specials are escaped in the same scan as the inline markers |
| `lib/markdown.js`, `renderMarkdown` | pushed blocks, then mapped them again to add `"\n"` and joined | pushes each block with its `"\n"` and joins once |
| `index.js`, `wordsOf` | `text.match(/\S+/g)` per body: an array of every word, twice over the whole corpus | a code-unit table of exactly the `\s` set, one pass, no allocation |
| `lib/template.js`, `renderNodes` | `xs.forEach` with a fresh closure per rendered list | a plain indexed loop |

## Measured

3 practice cases, 1070 Markdown files, 2.77 M characters, against the unmodified house baseline, on one
machine, alternating arms, 7 runs each, CPU time (user + system, all threads):

    baseline 1.09 s    this entry 0.72 s    1.5x

`node cli/makefun.mjs grade-local --kit <kit> --bot . --driver process` passes every case and reports the
answers byte-identical to the reference's.

## Layout

- `main.ts`, the competition driver, unchanged.
- `solve.js`, the entry: `run(input)` is the vendored build.
- `vendor/makefun-site/`, make.fun's own makefun-site 1.0.0 with its LICENSE and NOTICE kept beside it.
