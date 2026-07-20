# paper-proofread

[![Version](https://img.shields.io/badge/version-1.1.0-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-purple.svg)

[한국어](README.md) · **English**

A [Claude Code](https://claude.com/claude-code) skill that walks a Korean academic manuscript (`.docx` · `.pdf` · `.md` · `.hwp`) end to end in ~2,000-character chunks and returns a **corrected manuscript** plus an itemized **correction table**.
It works one layer below what a spell checker sees — the **sentence layer** (double passives, translationese, subject–predicate drift) and the **citation layer**, where in-text citations have to line up with the reference list.

<!-- demo GIF goes here -->

## What it checks

Four axes — **orthography** (spelling, spacing, loanword transliteration, number/unit consistency), **sentence editing** (subject–predicate agreement, double passives, translationese, redundancy), **logic & structure** (paragraph structure, terminology consistency, table numbering and whether the body references them), **citations & references** (in-text citations vs. the list, bibliographic completeness). English-language proofreading tools don't model double passives or translationese at all — those categories don't exist in the language they were built for — and Korean spell checkers stop at the orthography layer.
One more axis you can switch on. **Reference-existence verification** resolves DOIs and URLs for real and compares the retrieved metadata against the list. It's the most valuable axis in a final pre-submission pass on an AI-assisted manuscript — a plausible but nonexistent work often has *cleaner* bibliographic formatting than what a human types by hand, which is exactly why the eye doesn't catch it.

| # | Location | Sev. | Type | Original | Suggested | Reason |
|---|------|--------|------|------|--------|------|
| 1 | §2.1/C3 | 🔴 | Double passive | 분석되어지고 | 분석되고 | Doubled passive removed |
| 2 | §3.2/C5 | 🟡 | Translationese | ~에 있어서 | ~에서 | Unnecessary calque |
| 3 | References | 🔴 | Suspected hallucination | (list item 12) | — | DOI does not resolve · not in that volume/issue |

Illustrative. Every finding carries a severity — 🔴 error (must fix) / 🟡 suggest (recommended) / 🔵 style (optional) — and the location column (`§2.1/C3`) gives the section name and chunk number, so you can go straight to the spot in your original. Alongside this **correction table** you get the **corrected manuscript** and a **summary report** of recurring error patterns and terms found to be inconsistent.
→ [Why I built this · detailed usage](docs/why.md)

## Install

```bash
# A. git clone (recommended — update later with git pull)
mkdir -p ~/.claude/skills && cd ~/.claude/skills && git clone https://github.com/parkjui92/paper-proofread.git
# B. the .skill bundle (files only, no git)
mkdir -p ~/.claude/skills/paper-proofread && cd ~/.claude/skills/paper-proofread
curl -L -o paper-proofread.skill https://github.com/parkjui92/paper-proofread/raw/main/paper-proofread.skill
unzip paper-proofread.skill && rm paper-proofread.skill
# then restart Claude Code → if ~/.claude/skills/paper-proofread/SKILL.md exists, you're set
```

## How to use it

```
Proofread this paper before I submit it. The .docx is attached.  ← all four axes (non-papers too)
Also verify that the references actually exist                   ← reference verification on
Just spelling and spacing, quickly                               ← pick an axis · "conservatively" = 🔴 only
(triggers are Korean apart from "proofreading" — include that word, or invoke the skill by name)
```

| Parameter | Default | Description |
|---|---|---|
| `strictness` | moderate | `conservative` (🔴 only) / `moderate` (🔴🟡) / `aggressive` (🔴🟡🔵) |
| `focus` | full | `full` / `surface` (orthography) / `sentence` / `reference` |
| `verify_references` | off | On resolves DOIs and URLs for real (network required) |
| `chunk_size` · `citation_style` | 2000 chars · APA 7th | Chunk size (cut at paragraph boundaries) · citation format |

## Requirements & limits

- Claude Code (local) and claude.ai both supported. `.docx` · `.pdf` · `.hwp` extraction falls back automatically when a dependency is missing, and the results say which path was used. Manuscript text is never sent anywhere — the only step that touches the network is reference verification, if you turn it on, and even then only bibliographic data is queried. You can direct it in English; the manuscripts it checks, and the rule reference it works from, are Korean
- **It misses things, and it flags things wrongly.** The correction table is a review list, not a set of decisions. "Unverified" results from reference checking (Korean journals without DOIs, paywalled texts) are not errors either — they're the list a human needs to look at
- **It does not touch content.** Review at the argument and methodology layer belongs to the gates in [socsci-paper-kit](https://github.com/parkjui92/socsci-paper-kit). Further reading: [SKILL.md](SKILL.md) · [references/rules_ko.md](references/rules_ko.md) (14 translationese patterns and the rest of the Korean rule reference) · [CHANGELOG.md](CHANGELOG.md)

## Series

One piece of a "verification built into the research workflow" series. Three are kits that carry a document to completion with an agent team; one builds lecture decks and edits them live; four are standalone skills covering a single point in the workflow.

**Agent-team kits** — plugins that run design → review gate → research → drafting → review → conversion as a team

- [policy-research-kit](https://github.com/parkjui92/policy-research-kit) — policy research reports
- [rnd-proposal-kit](https://github.com/parkjui92/rnd-proposal-kit) — Korean government R&D proposals
- [socsci-paper-kit](https://github.com/parkjui92/socsci-paper-kit) — social science papers. **Uses this skill as a companion** — the team's finalizer calls paper-proofread to copyedit the approved draft

**Authoring kit** — a plugin that builds lecture decks and edits them live in the browser

- [lecture-deck-kit](https://github.com/parkjui92/lecture-deck-kit) — HTML lecture decks + live editing

**Standalone skills**

- [paper-proofread](https://github.com/parkjui92/paper-proofread) — Korean academic proofreading **(this repository)**
- [fact-verify](https://github.com/parkjui92/fact-verify) — source verification (4-tier classification, re-citation chain detection, URL/DOI existence checks)
- [form-tailor](https://github.com/parkjui92/form-tailor) — institutional document formats
- [report-to-brief](https://github.com/parkjui92/report-to-brief) — report compression

## License

[MIT](LICENSE)
