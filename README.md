# Build a static site faster — a faster makefun-site 1.0.0

[![bounty](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=makefun)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/haoye2023/makefun-site-generator-speed-entry/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=makefun)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/haoye2023/makefun-site-generator-speed-entry/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=makefun)

Entry for the make.fun bounty [Build a static site faster, 22 November 2026](https://make.fun/bounty/site-generator-speed-01-run-20261122).

`solve.js` is a dependency-free ES module whose `run(input)` returns, for every
input `{ site, templates, files }`, exactly the answer makefun-site 1.0.0
returns — the same output tree, byte for byte: every page's `index.html`, the
index and tag pages, `build-report.txt` and each static file, under makefun-site's
own front-matter, template, Markdown, slug and report rules. `main.ts` is the
harness driver: it reads every case from `/input/cases.json`, answers each with
`run`, and writes `/output/answers.json`.

Where the reference lexes each body into block tokens and then renders them,
walks every inline string one character at a time through a regex escape, and
recomputes each slug through NFKD and a Unicode property regex, this entry:

- renders Markdown in a single pass over the body's lines, with range-based
  inline rendering that moves whole plain runs with `slice` and only stops on
  a character that can change the output;
- gives slugs an ASCII fast path (NFKD is the identity on ASCII, and no
  combining mark is ASCII) plus a per-run memo, so a repeated title or tag
  name is slugged once per case — nothing is cached from one case to the next;
- renders templates with one flat scope object and save/restore around `for`
  loops instead of a chain of scope objects, which is the same binding the
  reference's chain gives (the innermost loop's names win, the outer ones
  come back when the loop ends);
- counts a body's words without building the array of all of them, and never
  builds token objects for Markdown blocks or a second copy of a post's card.

On the kit's practice set (3 cases, 257 / 348 / 486 files) it answers every
case identically to makefun-site 1.0.0 and to the kit's reference, and `run()`
itself takes a median of **166 ms** per 3-case set against the house
baseline's **551 ms** (about **3.3× faster**, measured in-process with
`process.cpuUsage`, Node 24); end to end, including node start-up and the
`cases.json` / `answers.json` round trip the driver does for both entries,
it takes about **0.69 s** of CPU against the baseline's **1.19 s**.
The same byte-identical result held on 12 further cases drawn from four fresh
seeds with the kit's own generator.

Run it the way the harness does:

    MAKEFUN_INPUT_DIR=/input MAKEFUN_OUTPUT_DIR=/output node main.ts

Determinism: the build reads no clock, no environment, no randomness and no
file, and every per-run table is created inside `run()`, so two executions
write byte-identical answers.
