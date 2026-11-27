---
name: stop-slop
description: |
  Edits prose to strip the tells that make writing sound machine-generated:
  throat-clearing openers, hedge words, binary contrast constructions, empty
  emphasis, and filler adverbs. Rewrites the text and scores it on directness,
  rhythm, trust, authenticity and density.
  Use when the user asks to "make this sound less like AI", "remove the AI tells",
  "de-slop this", "tighten this up", "edit my draft for tone", "does this read
  like a human wrote it", or "punch up this copy".
  Do NOT use for creating a new document from scratch (use docx), for slides
  (use pptx), or for drafting audience-targeted business updates from M365 data
  (use stakeholder-comms) — this skill edits prose that already exists.
metadata:
  category: writing
  icon: TextEditStyle
---

## Overview

This skill takes prose the user already has — an email, a post, a section of a
document — and rewrites it so it reads as though a person wrote it deliberately.
It removes recognisable generated-text patterns, then reports a score so the user
can see whether the edit actually landed.

The approach is adapted from the open-source "Stop Slop" skill by Hardik Pandya
(MIT licence), rewritten for Microsoft 365 use.

## When to Use

- The user pastes or points at a draft and wants it to sound more human
- A draft reads as generic, padded, or over-hedged and needs tightening
- The user asks for an edit pass on tone, rhythm, or wordiness
- The user wants to know whether a piece of writing reads as machine-generated

## When NOT to Use

- Creating a document, report, or SOP from nothing — use the `docx` skill
- Building slides — use the `pptx` skill
- Writing a stakeholder update assembled from calendar, email, or Teams data —
  use `stakeholder-comms`, then run this skill over its output if asked
- Translating, summarising, or fact-checking — this is an editing pass only

## Quick Start

```
User: "Make this email sound less like AI wrote it"
1. Read the source text (message body, input/ file, or M365 item)
2. Run the removal pass — phrases, structures, sentence-level rules
3. Rewrite, preserving every fact and the author's intent
4. Score the rewrite on five dimensions
5. Show the edited text, then the score, then the top three changes
```

## Core Instructions

### Step 1: Get the source text

Take it from the user's message when pasted. If they reference a file, read it
from `input/`. If they reference an email, document, or Teams message, retrieve
it with the relevant M365 tool (`outlook-GetMessage`, `ReadFileContent`,
`m365_teams-GetChatMessage`) before editing. Never edit from memory of a
described document.

### Step 2: Remove the phrase-level tells

Cut or replace:

- **Throat-clearing openers** — "In today's fast-paced world", "It's worth
  noting that", "Let's dive in", "At its core"
- **Empty emphasis** — "truly", "really", "incredibly", "absolutely",
  "game-changing", "powerful", "seamless", "robust"
- **Hedges that carry no information** — "arguably", "perhaps somewhat",
  "it could be argued that"
- **Corporate filler** — "leverage", "utilise", "synergy", "best-in-class",
  "at the end of the day", "circle back"
- **Meta-commentary** — sentences about the writing rather than the subject:
  "This section will explore…", "Here's the thing"
- **Adverbs doing a verb's job** — replace "moved quickly" with a stronger verb

### Step 3: Remove the structural tells

- **Binary contrast** — "It's not X, it's Y" / "This isn't just A — it's B".
  State the point once, directly.
- **Negative listing** — defining a thing by what it is not before saying what
  it is
- **Rhetorical setup** — asking a question purely to answer it in the next line
- **Rule-of-three padding** — three parallel clauses where one carries the meaning
- **Dramatic fragmentation** — one-word sentences for effect. Cut them.
- **False agency** — inanimate subjects performing human actions ("the data
  argues", "the report believes")
- **Passive voice** — rewrite to active unless the actor is genuinely unknown
- **Uniform paragraph length** — vary it; generated text is metronomic

### Step 4: Apply sentence-level rules

- Start sentences with a subject, not a Wh- word or a participle
- Replace em dashes with a comma, full stop, or restructure
- Avoid absolute extremes ("always", "never", "everyone") unless literally true
- Keep one idea per sentence; vary sentence length deliberately

### Step 5: Preserve meaning

Facts, figures, names, dates, commitments, and the author's position must survive
the edit unchanged. If a cut would drop a fact, keep the fact and cut the padding
around it instead. Flag anything ambiguous as `[check: …]` rather than guessing.

### Step 6: Score the result

Rate the rewrite 1-10 on each dimension and show the total out of 50:

| Dimension | Question |
|---|---|
| Directness | Does it state things, or announce that it will state them? |
| Rhythm | Do sentence lengths vary, or is the cadence uniform? |
| Trust | Does it respect the reader's intelligence, or over-explain? |
| Authenticity | Does it sound like a person with a view? |
| Density | Is anything still cuttable? |

Below 35/50, revise once more before showing the result.

## Output

Present in this order, as chat markdown — no file unless the user asks for one:

1. **The edited text**, ready to copy or send
2. **Score**: `Directness 8 · Rhythm 7 · Trust 9 · Authenticity 8 · Density 8 — 40/50`
3. **Top three changes**, one line each, naming the pattern removed
4. Word count before → after

If the user asks for the edit to be sent or saved, use the relevant tool
(`CreateDraftMessage` for email unless they explicitly say send) after showing
the text.

## Guardrails

- Never change facts, figures, names, or commitments to improve the prose
- Never send or post an edited message without showing it first; default to a
  draft when the source was an email
- Do not apply this to legal, contractual, or compliance text where exact wording
  is load-bearing — say so and stop
- Do not reproduce or rewrite third-party copyrighted text the user does not own
- If the source is already tight, say so and make no changes rather than
  inventing edits
- Show the score honestly, including when the rewrite scores low
