# MVP Spec — Search Recovery Assistant (simulated-dataset version)

This replaces any earlier "search your real Google Photos manually" version.
Build exactly this. Do not add screens, features, or complexity beyond what's
specified here — this is a graded graduation deliverable on a tight deadline.

## 1. What we're testing

The validated finding from 8 user interviews: people can describe a photo as
a multi-attribute memory (person + activity + place + time), a first search
on that memory comes back broad or noisy, and the product gives no help
figuring out what to try next — every recovery path users found (People tab,
timeline, a category, rewording, Gemini) was self-discovered, not suggested.

**This MVP tests: does surfacing "what I understood" + "what's missing" +
concrete next-step options, right after a weak result, help someone recover
faster than trial-and-error?**

## 2. Non-negotiable scope boundaries

- ❌ No login, no backend database, no real Google Photos API
- ❌ No image generation — use a small set of real or stock-style images, or
  simple styled cards with an emoji + caption if no images are available
- ❌ Don't build search for Express or Recognise gates — those aren't the
  hypothesis being tested
- ✅ A fixed, local `data/metadata.json` photo library (~25-30 items)
- ✅ Exactly 4 UI states (below) — no more
- ✅ Event logging to support Part 6/7 metrics (see section 6)

## 3. Data — `data/metadata.json`

Each entry:
```json
{
  "id": "photo_001",
  "emoji": "🚴",
  "caption": "Cycling along the coast",
  "date": "2023-11-14",
  "location": "Goa",
  "people": ["self"],
  "activity": ["cycling"],
  "objects": ["bicycle"],
  "event": "Goa trip",
  "kind": "photo"
}
```
Build the library to deliberately create **overlap** — several bicycle
photos, several Goa photos, several "self" photos — so that single-attribute
queries return too much, and only combined attributes narrow it down. This
mirrors the real interview evidence (Ashwani's bicycle photo, Chetna's Kerala
museum photo, Vibhuti's college farewell, Mansi's waterfall, Ankita's mother
photo). Reuse those five scenarios as the seed of the dataset, then pad with
distractor items sharing at least one attribute with each target so a
single-clue search is never enough on its own.

## 4. Retrieval engine (simple, deterministic — no AI needed here)

- **Any-match search:** an item matches if it shares ANY extracted attribute
  value with the query. This is what makes results "broad."
- **All-match search:** an item matches only if it shares ALL extracted
  attribute values. This is what a recovery step narrows toward.
- **Weak-result rule:** a result set is "weak" if any-match count > 5, or
  all-match count == 0. This triggers the Recovery Assistant (State 3).

## 5. The four UI states

**State 1 — Retrieval**
- One text input: "What are you trying to find?"
- Placeholder example: "That photo of me riding a bicycle during my Goa trip"
- [Find photo] button
- On submit → call AI clue extraction (section 7) → run any-match search →
  go to State 2

**State 2 — Results**
- Show the any-match result count and the grid of matching items
  (emoji/caption/thumbnail cards)
- If weak-result rule triggers: show a `[✨ Help me find it]` button
- User can also directly tap a card if they spot their photo (task complete)

**State 3 — Recovery Assistant** (only reached via the button above)
- "I understood:" — list the extracted attributes, each with ✓
- "I couldn't narrow down:" — list attribute types NOT mentioned in the
  original query (e.g. time, location) with a ?
- Present recovery options **only for attributes with more than one distinct
  value among the current any-match results** (no point offering to narrow
  by location if everything is already in Goa)
- Buttons: `[Narrow by <attribute>]` for each viable attribute, plus
  `[Describe something else]` (returns to State 1)

**State 4 — Refined results**
- Selecting a recovery option asks a follow-up (e.g. tapping "Narrow by
  time" shows the distinct time buckets present, user picks one) and reruns
  an all-match search including that new constraint
- Show new result count + grid
- If still weak, offer Recovery Assistant again (loop back to State 3) with
  remaining un-narrowed attributes
- If user taps their photo → task complete
- Cap at 3 recovery loops — if still not found after 3, show a plain
  "couldn't narrow further" state (this is itself useful data, not a bug)

## 6. Instrumentation (build from the start, not bolted on later)

Log these events to an in-memory array plus `localStorage`, each with a
timestamp and the session's task ID:
```
task_started        { query_text }
clues_extracted      { attributes }
initial_results       { count }
recovery_shown        { missing_attributes }
recovery_option_selected { attribute }
refined_results        { count }
task_completed         { photo_id, total_steps, time_ms }
task_abandoned          { reason }
```
Add a small "Export session log" button (visible always, e.g. footer) that
downloads/copies this event array as JSON — this is what testers send back
for Part 6 analysis. Don't build a dashboard for it; raw exportable JSON is
enough.

## 7. Where AI is actually used (only two calls, nothing else)

1. **Clue extraction** (State 1 → 2): natural-language query → structured
   attributes (person/activity/place/event/time/objects). Constrain the
   model to only pick from the vocabulary that exists in `metadata.json`,
   same approach as the current `index.html` (`ai()` function calling
   `window.claude.use("sample")`, with a keyword-matching fallback when that
   runtime isn't available — e.g. on GitHub Pages).
2. **Recovery framing** (State 3, optional): one short sentence like "I
   found 37 bicycle-related photos, but couldn't confidently narrow them to
   your memory" — can be templated with no AI call if that's simpler and
   more reliable; only use AI here if it clearly improves clarity.

Do not use AI for the retrieval logic itself (section 4) — that must be
deterministic so results are reproducible and testable.

## 8. File structure

```
/
├── index.html         # Single-file app (same pattern as current repo)
├── data/
│   └── metadata.json    # ~25-30 item library per section 3
├── README.md
├── PROJECT_CONTEXT.md
└── MVP_SPEC.md          # This file
```
Keep it a single-file frontend (no framework, no build step) — consistent
with the existing repo's architecture, per PROJECT_CONTEXT.md.

## 9. Definition of done

A tester who has never seen this project can: open the link, type a memory
in their own words, see a first result set, get stuck if it's broad, use the
Recovery Assistant to narrow it, find "their" photo, and have that whole
session exportable as a JSON log — with zero instructions beyond "try to
find something using this."
