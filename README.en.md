# paper-proofread

[![Version](https://img.shields.io/badge/version-1.1.0-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-purple.svg)

[한국어](README.md) · **English**

> A [Claude Code](https://claude.com/claude-code) skill that walks a Korean academic manuscript (`.docx` · `.pdf` · `.md` · `.hwp`) end to end in ~2,000-character chunks and returns a **corrected manuscript** plus an itemized **correction table**.
> It works one layer below what a spell checker sees — the **sentence layer** (double passives, translationese, subject–predicate drift) and the **citation layer**, where in-text citations have to line up with the reference list.

---

## Why I built this

The last pass over a manuscript before you hand it in has always been human work. The problem is that this particular job falls into a gap between the available tools.

Korean academic prose has failure modes of its own. Four that show up constantly:

**1. 이중피동 — the double passive.**
"분석되어지고 있다" (*"is being analyzed"*). The verb stem already takes `-되다`, which makes it passive; then `-어지다` is added, making it passive a second time. It's a norm violation, but it has become so common in Korean academic writing that it reads as perfectly fluent. Nothing snags when you read past it.

**2. 번역투 — translationese.**
"연구에 **있어서**" where plain "연구**에서**" ("in this study") says exactly the same thing in half the space. It's a calque — Japanese *~において*, English "with regard to" — that settled into Korean academic register and never left. Others in the same family: "~하는 것이 가능하다" (a literal rendering of "it is possible to," where Korean has a one-word form), "~로 사료된다" (an ornate hedge for "appears to be"), "~가 행해졌다" (a Japanese-shaped passive for "was carried out"). None of these are ungrammatical, so a spell checker passes every one. They just make sentences longer and claims vaguer.

**3. 주술 불일치 — subject–predicate mismatch.**
Korean is verb-final and lets you stack modifying clauses in front of the predicate without any structural limit. Subjects can also be dropped freely. Put those two together and a subject can end up far enough from its verb that agreement quietly breaks — and the author, who knows what the sentence was supposed to mean, is the last person who will notice.

**4. 띄어쓰기 — the spacing gray zone.**
Korean spacing has cases where two spellings are *both* standard. "시도해 보다" and "시도해보다" ("to try doing") are both correct. So the right answer isn't one of them — it's *whichever one you already used elsewhere in this manuscript*. A checker that flags one as wrong is worse than no checker at all. What you actually need is consistency detection across the whole document.

English-language proofreading tools don't model any of this. Not because they're weak, but because these categories don't exist in the language they were built for. Korean spell checkers do exist and handle the orthography layer well. What neither covers is the layer above it: sentence grammar, paragraph logic, terminology that drifts across a long document, and whether the citations in the body actually match the reference list. Which leaves reading the whole thing again, by hand.

And everyone knows what happens when a person reads a long manuscript in one pass. **The checking thins out toward the end.** You wrestle with every sentence in the introduction; by the conclusion your eyes are skating. Worse, you can't remember which of two legal spellings you picked twenty pages ago, so the same term ends up split across the document.

So this skill doesn't process a manuscript in one pass. It cuts it into roughly 2,000-character chunks and **checks them one at a time, in sequence.** Length goes up; the density of attention per chunk doesn't. What crosses the chunk boundary is a carried-forward record — abbreviation definitions, term spellings, table numbers, the running citation list — so a decision made early is applied consistently later.

That was version 1.0. Then a new problem showed up.

As **AI-assisted manuscripts** became common, a failure that used to be rare became ordinary: a plausible but **nonexistent work sitting in the reference list.** The author name reads naturally, so does the journal, so does the year, and the bibliographic formatting is often *cleaner* than what a human types by hand. Which is exactly why the eye doesn't catch it. Mismatches between body and list — a work cited in the text but missing from the list, or a work in the list that the text never actually calls — became more common for the same reason.

This isn't the kind of thing you fix by improving sentences. **You have to actually open the DOI and the URL.** That's why v1.1 added optional reference-existence verification (`verify_references`). It's the most valuable axis in a final pre-submission pass: a typo is a correction, but a citation to something that doesn't exist is a much heavier problem.

The sister skill [fact-verify](https://github.com/parkjui92/fact-verify) covers a different stretch of the same road. fact-verify screens sources at the **research stage**, before they ever reach the manuscript. paper-proofread checks, at the **manuscript stage**, the citations already embedded in sentences and whether they reconcile with the list. Filter once going in, cross-check once coming out.

---

## What it checks

Four axes, plus one you can switch on.

| Axis | What it looks at | Example |
|---|---|---|
| **Orthography** | Spelling, spacing, loanword transliteration, punctuation, number/unit consistency, abbreviations spelled out on first use | 데이타 → 데이터 (*data*) · 갯수 → 개수 (*count*) · 할수있다 → 할 수 있다 (*can do*, bound-noun spacing) |
| **Sentence editing** | Subject–predicate agreement, particle misuse, double passives, translationese, sentence length, redundancy, passive-voice runs, conjunction overuse | 분석되어지고 → 분석되고 (double passive removed) · ~에 있어서 → ~에서 |
| **Logic & structure** | Claim→evidence→wrap-up structure within paragraphs, transitions across chunks, terminology consistency, table/figure numbering and whether the body actually references them | 기술이전 in one section vs. 기술 이전 in another (same term, split spelling) |
| **Citations & references** | In-text citations vs. reference list, bibliographic completeness, format consistency, sort order, year agreement | (김철수, 2021) cited in the body but absent from the list |
| **Reference existence** (optional) | Resolves DOIs and URLs for real, checks identifier format, compares retrieved metadata against the listed title / authors / year | DOI fails to resolve → 🔴 suspected hallucinated citation |

Every finding carries a **severity**: 🔴 error (must fix) / 🟡 suggest (recommended) / 🔵 style (optional). The `strictness` option controls how much of that you see.

### Sample output — the correction table

Illustrative, to show the shape of the output.

| # | Location | Sev. | Type | Original | Suggested | Reason |
|---|------|--------|------|------|--------|------|
| 1 | §2.1/C3 | 🔴 | Spelling | 데이타 | 데이터 | Loanword transliteration standard |
| 2 | §2.1/C3 | 🔴 | Double passive | 분석되어지고 | 분석되고 | Doubled passive removed |
| 3 | §3.2/C5 | 🟡 | Translationese | ~에 있어서 | ~에서 | Unnecessary calque |
| 4 | §3.2/C5 | 🟡 | Redundancy | 약 30% 정도 | 약 30% | "approximately" stated twice |
| 5 | References | 🔴 | Citation mismatch | (김철수, 2021) | — | Cited in body, missing from list |
| 6 | References | 🔴 | Suspected hallucination | (list item 12) | — | DOI does not resolve · not in that volume/issue |

The location column carries the section name and chunk number, so you can go straight to the spot in your original.

---

## How it works

```
Read document (.docx / .pdf / .md / .hwp → text extraction)
  → Map structure · split into chunks (~2,000 chars, cut at paragraph boundaries)
  → [check chunk N → append to correction table → report back] × repeat
  → Reference check (in-text citations ↔ reference list)
  → ⚙ optional: reference-existence verification (resolve DOIs / URLs)
  → Consolidated output (correction table · corrected manuscript · summary report)
```

**Why chunks.** Processing a whole manuscript at once breaks two things: fine-grained errors get missed, and long documents run into context limits. Splitting keeps per-chunk scrutiny constant no matter how long the document gets.

**Then how does document-wide consistency survive?** State is carried across chunk boundaries:

- **Abbreviation definitions** — which chunk first spelled a term out (so later chunks use the short form)
- **Term spellings** — spacing and Hanja-gloss decisions for key terms
- **Auxiliary-verb spacing** — the convention observed at first occurrence, then applied throughout
- **Table/figure counters** — gaps in numbering, tables the body never references
- **Citation list** — every (author, year) seen in the body, collected for the final reconciliation against the reference list

Results are reported chunk by chunk as it goes, so you can intervene mid-run: "skip chapter 3," "leave that spelling, it's the journal's house style."

---

## Install

Install into Claude Code's user skills directory (`~/.claude/skills/`). Either method works.

### Option A — git clone (recommended)

Lets you update later with `git pull`.

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/parkjui92/paper-proofread.git
```

### Option B — download the `.skill` bundle

Files only, no git.

```bash
mkdir -p ~/.claude/skills/paper-proofread
cd ~/.claude/skills/paper-proofread
curl -L -o paper-proofread.skill \
  https://github.com/parkjui92/paper-proofread/raw/main/paper-proofread.skill
unzip paper-proofread.skill && rm paper-proofread.skill
```

Restart Claude Code and it's picked up automatically. If `~/.claude/skills/paper-proofread/SKILL.md` exists, you're set.

---

## How to use it

There are no commands to memorize — ask in natural language and the skill activates on any of these keywords:

> 교정 · 교열 · 교정교열 · **proofreading** · 논문 검토 · 논문 수정 · 원고 다듬기 · 맞춤법 검사 · 투고 전 검토 · 문장 교열 · 학술 글쓰기 점검

The trigger list is Korean apart from `proofreading`, so an English request should include that word. If it doesn't fire, invoke the skill by name.

> **Note on languages.** The manuscripts this skill checks are Korean — the rule reference (`references/rules_ko.md`) and the report templates in `SKILL.md` are written in Korean. You can direct it in English; the material it works on is not.

### Scenario 1 — Full pre-submission pass

```
Proofread this paper before I submit it. The .docx is attached.
```

Runs all four axes. Splits the manuscript into chunks, shows findings for each, then consolidates into one correction table plus the corrected manuscript. It pauses to report along the way, so you can steer as it runs.

### Scenario 2 — AI-assisted manuscript, right before submission

```
Also verify that the references actually exist
```

Turns on `verify_references: on`. After sentence-level editing finishes, it opens each DOI and URL in the reference list for real. Anything that fails to resolve, or whose retrieved metadata disagrees with the listed entry, is flagged 🔴 with "suspected hallucinated citation."

### Scenario 3 — In a hurry, pick an axis

```
Just spelling and spacing, quickly            → focus: surface
Sentences only — don't touch the citations    → focus: sentence
Check the references only                     → focus: reference
```

For when there's no time for a full pass, or when you've already corrected once and want to re-examine a single layer.

### Scenario 4 — Documents that aren't papers

```
Copyedit this policy report draft, conservatively
```

Not restricted to academic papers. Reports and proposals work too. The citation axis only means something if the document has citations — if it doesn't, exclude it with `focus`.

### strictness — how hard to push

| Situation | Say this | Result |
|---|---|---|
| Editing a co-author's manuscript, where rewriting someone else's voice is awkward | "conservatively" / "보수적으로 교정해줘" | `conservative` · 🔴 only |
| Ordinary pre-submission check | (don't specify) | `moderate` · 🔴 + 🟡 |
| Your own manuscript, style included | "go over the style too" | `aggressive` · 🔴 + 🟡 + 🔵 |

**Two passes work better than one.** Run `conservative` first and clear the 🔴s, which gets the manuscript normatively clean. Then run `aggressive` and work on style. Handed dozens of findings at once, you can't tell which ones are non-negotiable; split across two passes, the judgment is easy.

### What to do with 🔴 🟡 🔵

| Severity | Meaning | How to handle it |
|---|---|---|
| 🔴 **error** | Norm violation or citation mismatch — no judgment call involved | **Apply all of them.** If one looks wrong to you, it's usually the skill misreading context — point at that spot and ask |
| 🟡 **suggest** | Better if changed, but not incorrect as written | **Pick and choose.** Translationese is usually worth fixing, but keep it where a phrase is entrenched in your field or the author's voice depends on it |
| 🔵 **style** | Preference | **Handle by type, not item.** Rather than adjudicating each one, select a category: "just apply the sentence splits for anything over 60 characters" |

One principle worth stating outright: where two spellings are both correct (that "시도해 보다 / 시도해보다" case), a manuscript that is already internally consistent **is not flagged.** A recommendation is issued only when the two are mixed, and it points toward whichever form you used more. This is a tool for making a manuscript consistent, not for imposing a spelling preference.

It also leaves scholarly content and claims alone. Expression and formatting only.

### When to switch on reference-existence verification

Off by default — it's the only step that touches the network.

**Worth turning on**

- **Right before submission** — one pass where the sources actually get opened
- **Manuscripts where AI helped with the draft or the reference list** — this is where hallucinated citations actually appear
- **Co-authored or handed-over manuscripts** — when the list includes works you didn't read yourself
- **Manuscripts that sat for a while** — links go dead

**Not worth it**

- Environments with no network access
- When you only want a fast sentence pass — you're adding one lookup per entry in the list

Entries that can't be confirmed (paywalled, offline books) are **counted separately as "unverified" rather than declared wrong.** Keeping *failed to confirm* distinct from *does not exist* is the whole point of the step.

---

## Options

Picked up automatically from a natural-language request; you can also set them explicitly.

| Parameter | Default | Description |
|----------|--------|------|
| `chunk_size` | 2000 chars | Chunk size, cut at paragraph boundaries |
| `citation_style` | APA 7th | Citation format |
| `strictness` | moderate | `conservative` (🔴 only) / `moderate` (🔴🟡) / `aggressive` (🔴🟡🔵) |
| `focus` | full | Scope — `full` / `surface` (orthography) / `sentence` / `reference` |
| `verify_references` | off | On resolves DOIs and URLs for real (network required) |

---

## What you get

Three artifacts.

| Output | Contents |
|---|---|
| **Correction table** | All chunk-level findings consolidated: location, severity, type, original, suggestion, reason. Written to a separate `.md` file at 20 or more items |
| **Corrected manuscript** | Every chunk's corrected text stitched back in order. Markdown by default, `.docx` on request |
| **Summary report** | Total count and severity split, recurring error patterns, terms found to be inconsistent, citation-check results, structural recommendations |

The "recurring patterns" line in the summary is worth reading closely. Applying individual corrections fixes *this* manuscript; a line telling you that one particular calque keeps reappearing changes the next one.

---

## Requirements

- **Claude Code (local) or claude.ai** — both paths supported as of v1.1.
- Optional dependencies, with automatic fallback down the chain:

| Format | 1st | 2nd | 3rd |
|---|---|---|---|
| `.docx` | `pandoc` | `python-docx` | docx skill |
| `.pdf` | pdf skill | `pypdf` / `pdfminer.six` | — |
| `.hwp` · `.hwpx` | [kordoc](https://github.com/chrisryugj/kordoc) MCP | kordoc CLI | LibreOffice conversion |
| `.md` · `.txt` | read directly | — | — |

Whichever path was used is reported with the results.

> **`.hwp` / `.hwpx`** is Hangul Word Processor format — the de facto standard for Korean academic and government documents, and the reason a Korean-facing tool needs its own extraction chain.

> ⚠️ **Caveat on `.hwp` input.** The LibreOffice fallback can flatten tables and figures into plain text. Prefer the kordoc path for table-heavy manuscripts; if tables appear to have vanished from the extraction, the results will say so explicitly ("original comparison required").

## Manuscript privacy

Manuscript text is never sent anywhere. The only step that uses the network is reference-existence verification, and only if you turn it on — and even then, the only thing queried is bibliographic data (title, authors, DOI).

## Limitations

- **It misses things, and it flags things wrongly.** The correction table is a review list, not a set of decisions. A human decides what to apply.
- **It does not touch content.** Whether an argument is weak or a method is thin is not this skill's call. For review at that layer, use the gates in [socsci-paper-kit](https://github.com/parkjui92/socsci-paper-kit).
- **Reference verification will return a lot of "unverified."** Korean journals and monographs without DOIs, and paywalled full texts, especially. Unverified is not an error — it's the list a human needs to look at.
- Where the norm itself is split (auxiliary-verb spacing and similar), it doesn't impose one answer — it only checks **consistency within the manuscript**. An explicit journal style guide overrides it.

## Further reading

- [SKILL.md](SKILL.md) — the skill itself: the actual Phase 1–4 instructions and the full checklist
- [references/rules_ko.md](references/rules_ko.md) — the Korean editing rule reference: common misspellings, spacing rules, **14 translationese patterns**, preferred academic phrasing, APA 7th checklist
- [CHANGELOG.md](CHANGELOG.md) — version history

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
