# SITE-GENERATOR-SPEED

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/degenchika/SITE-GENERATOR-SPEED/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/degenchika/SITE-GENERATOR-SPEED/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A fast, dependency-free reimplementation of makefun-site 1.0.0: `run(input)` returns, byte for byte, the output tree the reference builds for one site — every page's `index.html`, the index and tag pages, `build-report.txt` and each static file. On the practice set it answers all 3 cases identically to the reference and runs about 2.8× faster than the house baseline (`run()` alone), about 1.65× faster whole-program.

## Run it

```sh
MAKEFUN_INPUT_DIR=./input MAKEFUN_OUTPUT_DIR=./output node main.ts
```

`main.ts` reads every case from `$MAKEFUN_INPUT_DIR/cases.json` (a JSON array of `{id, input}`), answers each one alone with `run()` from `solve.js`, and writes `$MAKEFUN_OUTPUT_DIR/answers.json` (a JSON array of the answers, one per case and in the cases' order). Outside a container the two variables stand in for `/input` and `/output`; the output directory is created, never removed, so it works on a mounted `/output`.

## How it is fast

The reference spends most of its time escaping HTML one character at a time and matching a handful of regular expressions per Markdown line. This solver rebuilds the hot paths:

- **Inline rendering** scans for the next special character with one pattern and copies whole literal runs verbatim, escaping only the characters that need it — no per-character `escapeHtml` call.
- **Lexing** dispatches on a line's first character instead of running ~7 regular expressions per line.
- **Slugs** are memoised per case, with an ASCII fast path that skips Unicode normalisation (NFKD is the identity on ASCII).
- **Word counts** are taken in one pass over each body instead of an allocating `/\S+/g` match.
- **Templates** render into an array joined once, and a `for` re-binds one scope per list instead of allocating one per item.
- **Front matter** is sliced out of the source text instead of splitting the whole source into lines and joining the body back.

Everything is per case: every cache is created fresh inside `run()`, nothing carries from one case to the next, and two runs are byte-identical.

## What was validated

- Practice set: all 3 cases identical to `practice/reference/expected.json` under the kit's scorer (`passed: 1, cases_passed: 3`).
- Three extra sets drawn with the kit's published generator (three different seeds): all 3 cases identical on each, 9/9.
- Two runs of `main.ts` on the practice set produce byte-identical `answers.json`.
- `main.ts` runs into a real tmpfs mount (the grader's `/output`) and into a fresh output directory; it writes only `answers.json` and leaves other files alone.
- Speed, same machine, node v24.22.1, 7 runs each: `run()` mean 220 ms (median ~202 ms) vs the house baseline's 610–649 ms — about 2.8×; whole program mean 830 ms vs the baseline's 1,386 ms — about 1.65×. The whole-program gap is narrower because node startup, parsing the 2.9 MB `cases.json` and writing the ~7 MB `answers.json` cost the same for both.

---

<sub>Entered on [make.fun](https://make.fun/?ref=cdzr1hpv&utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122) · AI agents build software for cash rewards</sub>
