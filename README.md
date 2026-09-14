# proptest-th

```
██████╗  ██████╗   ██████╗ ██████╗  ████████╗███████╗ ███████╗ ████████╗         ████████╗██╗  ██╗
██╔══██╗ ██╔══██╗ ██╔═══██╗██╔══██╗ ╚══██╔══╝██╔════╝ ██╔════╝ ╚══██╔══╝         ╚══██╔══╝██║  ██║
██████╔╝ ██████╔╝ ██║   ██║██████╔╝    ██║   █████╗   ███████╗    ██║     █████╗    ██║   ███████║
██╔═══╝  ██╔══██╗ ██║   ██║██╔═══╝     ██║   ██╔══╝   ╚════██║    ██║     ╚════╝    ██║   ██╔══██║
██║      ██║  ██║ ╚██████╔╝██║         ██║   ███████╗ ███████║    ██║               ██║   ██║  ██║
╚═╝      ╚═╝  ╚═╝  ╚═════╝ ╚═╝         ╚═╝   ╚══════╝ ╚══════╝    ╚═╝               ╚═╝   ╚═╝  ╚═╝
```

---

## ◆ PULSE

[![GitHub Pages](https://img.shields.io/badge/Pages-live-2ea44f)](https://suradet-ps.github.io/proptest-th/)
[![License](https://img.shields.io/badge/license-MIT%20OR%20Apache--2.0-blue.svg)](#-anatomy)

A property has a counter-example, and a minimal one - proptest-th is
the Thai bridge to that exact moment. This is the complete Thai
translation of the official Proptest book: 30 chapters built with
mdbook, terminology locked by a single glossary, and every code block
byte-identical to the original. The links are checked against the
built book (225 anchors), the structure mirrors the upstream repo
file-for-file, and the license travels with the text. Built for the
Thai-speaking Rustacean:
[suradet-ps.github.io/proptest-th](https://suradet-ps.github.io/proptest-th/).

| 30 chapters translated ▣ | Glossary ▣ | Links 225/225 ▣ | Build passing ▣ |
|---|---|---|---|

*v1.0.0 - translation, glossary, verification, and the static build
are all sealed.*

> Built with mdbook 0.5 + Markdown, translated from
> [proptest-rs/proptest](https://github.com/proptest-rs/proptest),
> verified by script and rendered as static HTML - a book with the
> shrinks on the page.
>
> **suradet-ps**, artifact keeper

---

## ◆ IGNITION

One runtime, three commands.

```
⟫ git clone https://github.com/suradet-ps/proptest-th.git
⟫ cd proptest-th
⟫ cargo install mdbook
⟫ mdbook serve book --open
```

Open [http://localhost:3000](http://localhost:3000).

```
⟫ mdbook build book                             # static HTML into book/book
⟫ powershell scripts/check-links.ps1            # all anchors in the built book (pwsh on Linux/macOS)
⟫ powershell scripts/verify-translation.ps1     # byte-exact check vs upstream
```

> On Linux or macOS, run the verification scripts using `pwsh scripts/<script>.ps1`.
> `verify-translation.ps1` checks against `proptest-rs/proptest` in adjacent directories or via `-Orig <path>`.

<details>
<summary>Translating a chapter</summary>

A chapter is a file: `book/src/<chapter>.md`, listed in
`book/src/SUMMARY.md`. The glossary lives in `GLOSSARY.md` - a term
is chosen once and reused everywhere. Code blocks, commands, links,
and filenames stay verbatim; only prose and headings are translated.
Heading anchors follow mdbook's slug rules (Thai tone marks are
stripped), so anchors are copied from the built HTML, never guessed.

</details>

---

## ◆ ANATOMY

One stack, zero custom JS, several quiet helpers.

- **Translates** - the complete book: introduction, getting started,
  the 14-chapter tutorial from strategy basics to configuration,
  failure persistence, forking, `no_std`, wasm, state machine
  testing, and the `proptest-derive` modifier and error references -
  Thai prose over untouched code.
- **Glossaries** - `GLOSSARY.md` locks the vocabulary (property
  testing = การทดสอบพร็อพเพอร์ตี, strategy = กลยุทธ์, shrinking =
  การชริงก์), so chapter nine agrees with chapter two.
- **Verifies** - `scripts/verify-translation.ps1` diffs every code
  block (115 of them), heading level, and link target against
  upstream `proptest-rs/proptest` - byte-exact or it does not pass.
- **Checks** - `scripts/check-links.ps1` walks the built book and
  resolves every anchor link against real heading ids - 225 of them,
  all reachable.
- **Builds** - mdbook renders static HTML into `book/book/`, zero
  server runtime, readable offline and searchable by built-in static
  index.
- **Licenses** - MIT OR Apache-2.0, inherited from upstream, with the
  LICENSE files shipped beside the text.

---

## ◆ RITUALS

**The core ceremony** - the translation pass:

1. Open a chapter in `book/src/`. The upstream `proptest-rs/proptest`
   repo sits beside it (clone
   `https://github.com/proptest-rs/proptest` alongside `proptest-th`)
   - structure is a contract.
2. Translate the prose; keep every code block and command as the
   original wrote it.
3. Consult `GLOSSARY.md` for every term that already has a canon.
   New terms get proposed in the glossary first.
4. Build, verify, check. The book builds clean, the diff is
   byte-exact, and the anchors resolve.

**The ceremony of the anchor** - mdbook slugs strip Thai tone marks
and vowel signs (`ที่ถูกต้อง` becomes `ทีถูกตอง`). Anchors are read
from the built HTML, written into the source, and re-verified - a
guessed anchor is a broken link waiting to happen.

**The ceremony of the code block** - a translated command that is not
byte-identical to the original is a regression, not a translation.
The verifier is the conscience of the repo.

---

## ◆ ECHOES

**Where this artifact is heading**

```
P1 ▸ SUMMARY + introduction, indexes ──────────────────────────────── ▸ sealed
P2 ▸ the proptest guide, getting started to state machine ─────────── ▸ sealed
P3 ▸ all 14 tutorial chapters ─────────────────────────────────────── ▸ sealed
P4 ▸ proptest-derive getting started, modifiers, error index ──────── ▸ sealed
P5 ▸ glossary, license, link verification, mdbook build ───────────── ▸ sealed
```

**Raising the artifact** - the honest path lives in `GLOSSARY.md`
(term canon), `scripts/` (the verification gate), and
`book/book.toml` (book config). New chapters follow the
frontmatter-free contract of the SUMMARY. Open an issue first to
discuss a change.

**Status** - on every change: `mdbook build book` must pass, the
translation verifier must report byte-exact code blocks across all 31
files (30 chapters + `SUMMARY.md`), and the link checker must report
`ALL ANCHOR LINKS OK`.
[Watch the gates](scripts).

---

```
  ─────────────────────────────────────────
   ทุกพร็อพเพอร์ตีมีกรณีที่ล้มเหลวของมัน
   ทุกหนังสือมีหน้าแรกของมัน
  ─────────────────────────────────────────
```

Translated from the [Proptest](https://github.com/proptest-rs/proptest)
book, which is licensed under [MIT](LICENSE-MIT) OR [Apache-2.0](LICENSE-APACHE).
