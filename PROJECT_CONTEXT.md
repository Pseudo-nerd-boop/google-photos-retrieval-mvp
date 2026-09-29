# Project Context — Search Recovery Assistant

Read this before making any changes. This is a graduation-project deliverable
with a specific evidence-to-build story that the code must keep matching.
Deadline-sensitive — prefer small, safe changes over rewrites.

## 1. What this project is

A Product Manager graduation brief: "increase the percentage of Google Photos
users who successfully retrieve a photo they remember but cannot precisely
describe." The full project has 8 parts; this repo is the **Part 5 deliverable
only** — an AI-native MVP. Parts 1–4 (research) already happened elsewhere and
produced the problem this MVP is built to solve. Do not re-derive the problem
from scratch — treat section 3 below as settled fact.

## 2. Research chain that led here (for context only, not to redo)

1. **Discovery engine** — 60 public posts (Reddit, Google Photos Help
   Community, Play Store reviews) about failed photo retrieval, collected and
   tagged. 11 were duplicates of the same source post; 49 unique episodes
   remained.
2. **Metric decomposition** — the business metric "successful retrieval of a
   vaguely remembered photo" was split into four gates a search has to pass:
   **Express → Understand → Recognise → Refine → Retrieval.**
3. **8 live user interviews** — participants with 4–5 years' Google Photos
   tenure and ~2,700–11,000 library items were each given a real retrieval
   task in their own library. Result: 8/8 eventually found their photo (no
   outright failures, unlike the public complaints). But 5/8 needed real
   effort, and in every one of those cases the trigger was the same pattern:
   a multi-attribute memory (e.g. "me riding a bicycle on a trip") produced
   results broader or noisier than expected. All 8 recovered afterward using
   a self-discovered workaround — People tab, timeline scrolling, a category
   like Screenshots, rewording keywords, or switching to Gemini — and in
   every case the product itself suggested none of these paths.

## 3. Locked problem statement (do not change without being asked)

> The first search is not the main failure. Users can already get a rough
> result from a multi-attribute memory. The real problem sits right after a
> weak result: nothing tells them why it was broad, wrong, or empty, and
> nothing points them to the next useful step. Each recovery path is
> something users discover on their own, not something the product offers.

**Target segment:** experienced Google Photos users (4–5 yrs tenure,
~2,700–11,000 items) who remember several attributes of a photo but rarely
its date.

## 4. What the MVP does, and why it's built this way

The MVP is a **recovery workflow that sits alongside the user's real Google
Photos** — it does not search photos itself and has no access to anyone's
library. That's a deliberate scope decision, not a shortcut:

1. User describes a half-remembered photo in plain language.
2. The app extracts structured clues (people / activity / place / context /
   time / visible text) and gives an exact first query to type into Google
   Photos.
3. User tries it in their own Photos app and reports one outcome: too many
   results, wrong people/things, nothing, can't tell which, or found it.
4. On a miss, the app diagnoses why and suggests up to 3 different next
   steps, each naming where to go (Search bar / People & pets / a category /
   Timeline) and what to try — never repeating an earlier query.
5. Repeats until found. A session log (memory, clues understood, every
   attempt, and steps-to-recovery) is shown at the end for the user to copy
   to the researcher.

**Intelligence is deliberately scoped to two of the four gates only:**
- **Understand** — turning a natural-language, multi-attribute memory into
  structured clues and a first search.
- **Refine** — diagnosing *why* an attempt failed and proposing a genuinely
  different next step, using the history of prior attempts so it doesn't
  repeat itself.

Express was not a target (8/8 interviewees could already express a memory
fine). Recognise was not built around (situational in the interviews, not a
consistent blocker). Do not add features that address Express or Recognise
without checking with the project owner first — it would break the
evidence-to-build story on the deck.

## 5. Current file structure

```
/
├── index.html          # The entire MVP — single self-contained file.
│                        # No build step, no external dependencies except
│                        # the runtime AI call described below.
├── README.md            # Public-facing repo description
└── PROJECT_CONTEXT.md    # This file — for agent/developer context only,
                           # not meant for end users
```

`index.html` is intentionally a single flat file (vanilla HTML/CSS/JS, no
framework, no bundler). Keep it that way unless explicitly asked to
restructure — simplicity here is a feature for a graduation deliverable that
needs to "just work" when a reviewer opens it.

### Internal structure of index.html
- Inline `<style>` — theme via CSS variables, light/dark aware
- Inline `<script>`, single `S` state object driving a `draw()` re-render
- `ai(prompt)` — calls `window.claude.use("sample")` when available (Claude
  artifact runtime only) and falls back to `fbClues()` / `fbNext()` keyword
  logic when not. **This AI call only works inside a claude.ai artifact
  view** (see section 6) — it will not work once deployed standalone on
  GitHub Pages. That's expected, not a bug: the fallback keyword mode is the
  correct behavior for the standalone deployment.
- `understand()` — Understand-gate logic
- `report()` / `fbNext()` — Refine-gate logic (diagnosis + next steps)
- `summary()` — builds the copyable session log for researcher use

## 6. Important constraint: two deployment targets, different capabilities

This MVP is meant to exist at **two links**, and they behave differently on
purpose:

1. **Claude artifact** (canonical, full-featured):
   https://claude.ai/artifact/YQPBb4RgFQV3bTXNTtzEet
   — AI clue-reading and next-step diagnosis work here via the in-artifact
   `window.claude` runtime.
2. **GitHub Pages** (this repo, code-visible for reviewers):
   — same file, but the AI runtime isn't present, so it automatically runs
   in keyword-fallback mode. This is a known, accepted limitation — do not
   try to wire up a real API key or backend for this deployment. Adding a
   real LLM API call here would require a server to hold the key safely,
   which is out of scope for this deliverable.

When deploying with GitHub Pages: just serve `index.html` as a static file
from the repo root, branch `main`. No build process needed.

## 7. Things not to do without checking first

- Don't add a backend, database, or API key — out of scope for this MVP.
- Don't rewrite the problem statement, target segment, or gate framing in
  README.md or in-app copy — it must match the deck being built alongside
  this repo.
- Don't add features targeting Express or Recognise gates.
- Don't convert this to a framework (React etc.) or add a build step unless
  explicitly asked — single-file simplicity is intentional here.
- Don't remove the fallback keyword mode — it's required for the GitHub
  Pages deployment to be usable at all.

## 8. What's genuinely open to improve

- Visual polish / styling
- Expanding the fallback keyword vocabulary for better offline matching
- Adding more "where to go" suggestion types if evidence supports it (check
  with project owner — would need to trace back to interview evidence)
- Better mobile responsiveness
- Accessibility (labels, contrast, keyboard nav)
