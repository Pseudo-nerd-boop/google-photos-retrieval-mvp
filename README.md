# Search Recovery Assistant — AI-Native MVP

Part of the Google Photos Retrieval graduation project (Part 5).

## Live prototype
https://claude.ai/artifact/YQPBb4RgFQV3bTXNTtzEet

## What it is
A retrieval-recovery workflow used **alongside a person's own Google Photos**.
It does not search photos itself — it reads a half-remembered, multi-attribute
memory, gives the user an exact first search to try in Google Photos, and when
that search misses, diagnoses why and suggests a different next step
(People tab, a category, the timeline, or a reworded search).

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
  multi-attribute memory (Understand), and diagnosing a miss to suggest a
  different next step (Refine).
- The actual searching happens in the user's real Google Photos, so the
  retrieval task is real, not a simulated dataset.
- This is a semi-manual MVP by design: the assistant relies on the user
  reporting what came back, since it has no access to anyone's photo library.

## Running it
Open `index.html` in a browser. No build step, no dependencies — a single
self-contained file. AI-based clue reading and next-step suggestions require
opening it as a claude.ai artifact (see live link above) while signed in;
opened as a plain local file, it falls back to basic keyword matching.

## Limits
- Not connected to the Google Photos API — the user manually reports outcomes.
- Suggested next steps are limited to: Search bar, People & pets, a
  category (e.g. Screenshots), and the Timeline.
- Tested with a small, single-network sample (8 interviewees); see the
  project deck for full methodology and limits.
