# Search Recovery Assistant — AI-Native MVP

Part of the Google Photos Retrieval graduation project (Part 5).

## Live prototype
https://claude.ai/artifact/YQPBb4RgFQV3bTXNTtzEet

## What it is
A self-contained retrieval-recovery prototype with its own small simulated
photo library (`data/metadata.json`), so anyone can attempt a full retrieval
task with no setup — no Google account or personal photo library needed. The
user describes a half-remembered, multi-attribute memory; the app extracts
structured clues, searches the library, and — if the first result is too
broad or empty — surfaces a Recovery Assistant showing what it understood,
what's missing, and concrete next steps to narrow down to the target photo.

## Where this comes from
Built from Part 1–4 of this project:
- 49 unique public episodes (Reddit, Google Photos Help Community, Play Store)
- A four-gate metric decomposition: Express → Understand → Recognise → Refine
- 8 live user interviews

All 8 interviewees eventually found their target photo, but 5 of 8 needed
real effort, and every recovery (People tab, timeline scroll, category
browsing, rewording, switching to Gemini) was self-discovered — the product
never suggested it. That is the specific gap this MVP targets.

## Why it's built this way
- Intelligence is used only where the research pointed to it: reading a
  multi-attribute memory into structured clues (Understand), and diagnosing
  a broad/empty result to suggest a genuinely narrowing next step (Refine).
- The retrieval engine itself is deterministic (attribute matching), not AI
  — this keeps results reproducible and testable.
- Every step is logged and exportable as a JSON session log, used as Part 6
  testing evidence.

## Running it
Open `index.html` in a browser (with `data/metadata.json` alongside it). No
build step, no dependencies. AI-based clue extraction requires opening it as
a claude.ai artifact while signed in; elsewhere (e.g. GitHub Pages), it falls
back to keyword matching automatically.

## Limits
- Uses a small simulated library (~25-30 items), not a real photo backend.
- Recovery suggestions are limited to attributes present in the dataset
  (person, place, activity, time, event, objects).
- Tested with a small, single-network interview sample (8 people); see the
  project deck for full methodology and limits.

See `MVP_SPEC.md` for the full product and technical specification.
