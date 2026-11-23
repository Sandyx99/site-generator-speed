# site-generator-speed — a faster makefun-site

[![make.fun competition](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/competitions/site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/Sandyx99/site-generator-speed/rank.svg)](https://make.fun/competitions/site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/Sandyx99/site-generator-speed/reward.svg)](https://make.fun/competitions/site-generator-speed-01-run-20261122)

A fast, dependency-free reimplementation of [make.fun](https://make.fun)'s
makefun-site 1.0.0 static site generator, built for the
[Build a static site faster, 22 November 2026](https://make.fun/competitions/site-generator-speed-01-run-20261122)
competition.

`main.ts` reads every case from `/input/cases.json`, answers each one with `run()`
from `solve.js`, and writes `/output/answers.json`. `solve.js` returns the same
output tree makefun-site 1.0.0 returns for the case's site — every page's
`index.html`, the index and tag pages, `build-report.txt` and each static file,
byte for byte — but faster: about 3× the house baseline's CPU time on the
practice set (measured beside it on the same machine).

How it is faster, without changing an answer:

- Markdown is lexed and rendered in one pass, with no token objects; inline
  markup is rendered by a single scan whose plain runs are escaped in bulk,
  and whose closers are found with `indexOf` (backslash-escape parity kept).
- `slugify` takes an ASCII fast path (NFKD + mark-strip is the identity on
  ASCII) and is memoised per case; front matter finds its close line with
  `indexOf` instead of splitting the whole body.
- Templates are interpreted over a flat scope, with includes memoised on the
  values of the included template's free names — so every post's card and the
  constant header/nav are rendered once per case — and templates whose free
  names are only `site`, `tags` and `recent` are rendered once per case.
- Every cache is created fresh inside `run()`, so one case never affects
  another; the build is deterministic and dependency-free.

## Layout

- `main.ts` — the driver (reads `/input/cases.json`, writes `/output/answers.json`).
- `solve.js` — the fast implementation; the only file that changed from the
  house baseline starter besides this README.
- `vendor/makefun-site/` — makefun-site 1.0.0, the reference program, kept
  untouched for comparison (see `SOURCE.txt`).

## Grade locally

From the unpacked competition kit:

```sh
node cli/makefun.mjs grade-local --kit . --bot <this repository> --driver docker
```

(`--driver process` where docker is unavailable.) The practice set scores
3/3 cases identical, and the L1 regrade reproduces byte for byte.
