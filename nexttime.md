# Where things stand — testmessj

Resume by starting Claude in `/home/v/git-linux/testmessj` and saying *"read
nexttime.md"*. `CLAUDE.md` holds the invariants and the traps; this file holds
the state of play and what is still open.

Written 2026-09-20, at commit `c3a28dc`.

## What this is

`testmessj` is a browser rewrite of the Python `testmess`: one multiple-choice
test in Word goes in, any number of shuffled variants come out, each written as
a student copy and — where the source has an answer key — a professor copy.
Everything happens in the tab; nothing is uploaded.

- Live: <https://zerogvt.github.io/testmessj/> (GitHub Pages, deployed by
  `.github/workflows/deploy.yml` on push to `main`, with `npm test` as the gate)
- Local: `/home/v/git-linux/testmessj`, `main` in sync with origin
- The Python original: `/home/v/git-linux/testmess` (untouched except for one
  branch, below)

## Where the code lives

| | |
|---|---|
| `src/markers.ts` | label patterns, `KEY_WORDS`, `sameMarker` |
| `src/xml.ts` | paragraph markup: text, labels, relabelling |
| `src/zip.ts` | the archive layer, hand-written on `CompressionStream` |
| `src/parse.ts` | `parseExam`, `validateExam`, `Exam.hasKey` |
| `src/variants.ts` | the shuffle and its seeded generator |
| `src/render.ts` | rebuilding `word/document.xml`, writing packages |
| `src/metadata.ts` | scrubbing author details; warning about comments |
| `src/i18n.ts` | every user-visible string, English and Greek |
| `src/errors.ts` | `AppError`/`ExamError`, code + params, never a sentence |
| `src/main.ts` | the page |
| `tests/` | 308 tests, `npm test` |

Three samples in `samples/`, which is also the build's `publicDir`, so they are
served for download under their own names: Latin-lettered, Greek-lettered, and
Greek with no key page.

## What this session did, in order

1. **Ported the whole thing to TypeScript** — zero runtime dependencies: the
   browser supplies the ZIP codec, the XML parser and the serialiser. Vite,
   Vitest, TypeScript and jsdom are tooling only.
2. **Published it to GitHub Pages** and walked through enabling Pages by hand.
3. **A privacy pass**: papers are scrubbed of author metadata by default;
   comments and tracked changes are warned about, never silently rewritten; a
   `<meta>` CSP (`default-src 'none'`, `connect-src 'self'`); zip-bomb caps;
   the download blob released on `pagehide`; Actions pinned to commit SHAs.
4. **An unmissable notice**: a static banner plus a modal acknowledged on every
   visit (nothing stored), backed by an MIT `LICENSE`.
5. **Removed the owner's name** from the samples and, by rewriting all three
   commits and force-pushing, from this repository's history.
6. **Dropped the credit line** to the original project from the page footer.
7. **Made the samples downloadable**, so the expected Word layout can be seen.
8. **Greek as well as English**, with flag buttons, `?lang=el`, translated
   errors, and a Greek-aware key heading.
9. **A test with no answer key** is accepted and yields student copies only.
10. **Step 1 redesigned**: a green/red card under the picker — file name first,
    then what was read, then what it means — with the first-timer advice folded
    into a `<details>`.
11. **Every spelling of the key heading** recognised (`Answers`, `Key`, `Keys`,
    `Λύσεις`, `ΑΠΑΝΤΗΣΕΙΣ`, …), guarded by whole-paragraph matching and by
    only accepting a heading after the questions.

## Still open

- **Nobody has opened a generated paper in Word.** The package writer is new
  code; the checks are structural. Do this before a class sees a paper.
- **The Python repo has an unmerged branch**: `scrub-sample-metadata` (pushed,
  two commits — the author name out of its samples, and the NTFS stream files
  untracked). The PR was never opened because `gh`'s token is stale; open it at
  <https://github.com/zerogvt/testmess/pull/new/scrub-sample-metadata>. Body
  text was drafted in the session log.
- **`gh auth login` is worth re-running** — pushes work (git credentials are
  fine) but every API call fails with 401, so no `gh pr create`, no `gh run
  list`.
- **History caveats from the rewrite**: GitHub may still serve the orphaned
  commit `66f238a` by direct SHA until it garbage-collects; support will purge
  it on request. The commit *author* on every commit is still the owner's real
  name and email — deliberately left. Old `testmess` releases still contain the
  unscrubbed samples, since their zips are built from their tags.
- **Usage stats** are wanted in a future iteration. Three things have to move
  together: the CSP's `connect-src 'self'`, the page's "no analytics, no
  cookies, no stored state" promise, and the same claim in `README.md` and the
  notice.
- **`Answer sheet` is deliberately not a key heading** — in most papers that is
  the blank sheet a student writes on. Easy to add if wanted.
- A `samples/calculus_practice_test_2_lower.docx` (lowercase markers `a)`–`d)`,
  lowercase key) appeared mid-session and was deleted on request. No sample
  currently covers lowercase markers, though `sameMarker` handles them.

## How this work was verified

Beyond `npm test`, two things that are worth repeating and easy to forget:

- **The original Python implementation reads the papers back.** From
  `/home/v/git-linux/testmess`, `import testmess as tm` plus the helpers in
  `test_testmess.py` (`document_root`, `math_signature`, `count_math`,
  `paragraph_texts`) give an independent check: a second ZIP reader, a second
  parser, and an equation comparison that ignores namespace prefixes.
- **The built page is driven in real Chromium** over the DevTools protocol —
  no Playwright, no new dependency. See `CLAUDE.md` → Verification.

Both caught real bugs this session: the double XML declaration (Chromium's
serialiser writes its own), and Escape dismissing the notice despite a
prevented `cancel`.
