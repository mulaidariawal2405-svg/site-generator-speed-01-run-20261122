# site-generator-speed-01-run-20261122

A faster drop-in for makefun-site 1.0.0: `solve.js` answers every case
exactly as makefun-site does, and `main.ts` is the harness driver.

## How it is faster

- Single-pass Markdown renderer: no lex tokens, no regex per line, and
  range-based inline rendering (strong, emphasis and link texts are never
  copied into substrings). A shared regex jumps over plain runs.
- Cached `slugify` with a pure-ASCII fast path (ASCII is already NFKD and
  holds no combining marks) and cached long dates; titles, tag names and
  headings repeat a lot inside one case.
- Hand-rolled front matter parser (no per-line regexes).
- A tight template tree interpreter that appends to one output array
  instead of concatenating strings per node.
- No state is kept between cases: every cache is created fresh in `run()`.

## Checks

Practice set: 3/3 cases identical to the reference (official scorer:
`passed: 1`), two driver runs byte-identical. 48 freshly generated sites
from the kit's own generator (16 seeds x 3 size bands) plus 11 hand-crafted
edge inputs (broken front matter, broken templates, unclosed fences,
Unicode slugs, shadowed statics, CRLF, self-including templates) all
identical to the reference build.

CPU time on the practice set (3 cases, this machine): about 0.55 s cold
per fresh process vs about 1.2 s for the house baseline, and about 0.21 s
steady-state vs about 0.69 s.
