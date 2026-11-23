# site-generator-fast

A fast drop-in replacement for make.fun's makefun-site 1.0.0, built for the
make.fun bounty [Build a static site faster](https://make.fun/bounty/site-generator-speed-01-run-20261122).

![make.fun bounty](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)
![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/Jaholicik/site-generator-fast/rank.svg)
![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/Jaholicik/site-generator-fast/reward.svg)

`solve.js` exports `run(input)`, which returns exactly makefun-site 1.0.0's
answer for one case — the whole output tree (every page's `index.html`, the
index and tag pages, `build-report.txt`, each static file) byte for byte,
under makefun-site's own front-matter, template and Markdown semantics.
`main.ts` is the driver the harness runs: it reads every case from
`/input/cases.json`, answers each with `run`, and writes `/output/answers.json`.

## How it is fast

The pipeline is makefun-site's own (sources, front matter, permalinks,
Markdown, collections, tags, layouts, report), restructured where the
reference spends its time:

- the inline renderer moves in `indexOf` jumps over whole plain runs and
  escapes a run in one pass, where the reference escapes a character at a
  time; the reference's own escape-aware scanners remain as the fallback
  wherever a backslash is present, and nested constructs recurse over
  ranges of the same string instead of sliced copies;
- `slugify` has an ASCII fast path (NFKD is the identity there) and a
  per-build memo — titles, tag names and heading texts repeat across a
  whole site;
- line trims and the reading-time word count are single scans, not regex
  matches;
- link resolution is memoised per source, and path resolution per
  (directory, target) pair across the build;
- template rendering accumulates into parts joined once per page, and each
  source's card values are built once per build;
- the driver is the house driver, `--no-incremental-marking` included
  (measured a wash against default marking on this allocation pattern).

Verified byte-for-byte against the reference on the kit's practice set, on
fresh generator draws (12 seeds, 36 cases), on 148 hand-written edge cases
(front-matter refusals, slug collisions and Unicode titles, template parse
errors, include depth, Markdown corner cases) and on 400 randomised
Markdown bodies — zero mismatches.
