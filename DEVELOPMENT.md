# Chord Diagram Generator - Development Guide

## Project Overview
A single-file, no-build-step web app that generates interactive SVG guitar chord diagrams with accurate music theory notation, audio playback, filtering, and PNG export.

## Key Files
- `chord-diagrams.html` — the entire app (data + UI + logic)
- `chord-diagrams copy.html`, `chord-diagrams copy 2.html` — pristine backups, used to recover from bad bulk-edit bugs. Do not overwrite.
- `DATA_SCHEMA.md` — full voicing-array field reference
- `PIPELINE.md` — how bulk data edits are made

## Architecture

### Data Structure
See `DATA_SCHEMA.md` for the full 15-field voicing array. Top-level shape:
```js
const chordData = {
    standard: {
        'A': [ [voicing1], [voicing2], ... ],
        'A#': [ ... ],
        // ... one key per chord root/name grouping
    }
}
```

### Note Notation (Interval-Based)
Notes are derived from intervals, not fixed enharmonic tables, to guarantee correct spelling:
- Each interval maps to a scale-degree letter (1→root's letter, 3→a letter two up, 5→a letter four up, etc.), then an accidental is applied.
- Rule: never reuse the same letter for two different scale degrees within one chord.
- Example: Aaug → A, C#, **E#** (not F) — the #5 must be spelled as a raised E, not reinterpreted as F.

### Barre Detection
A barre requires the barre fret to actually be shared by 2+ **contiguous** strings — not just any 2 strings at the same fret with a gap between them (that's a coincidental match, not a real barre). Shape sub-labels (`positionLabel`, index 9): E-shape barre, A-shape barre, D-shape barre, Other barre.

### Difficulty Rating
See `DATA_SCHEMA.md` — computed from type + fret span (frets *touched*, inclusive count) + finger count, since raw span alone doesn't track real-world playability (power/barre chords span several frets but are easy/medium; some `fretted` voicings are hard despite a small span due to interrupted partial-barre finger patterns).

### Key Signatures
18 keys (9 major + 9 relative minor). `getKeysForChord()` does strict all-notes-in-scale matching; `formatKeysDisplay()` adds a second, more permissive tier — see `DATA_SCHEMA.md` for the exact two-tier logic and why it exists.

### Sort Order
Card display order (within a root note) is a two-tier scheme: major/power/minor triads first, then everything else grouped by chord family (7th, sus2, sus4, altered-bass, catch-all) and ranked within each family by a numeric complexity score. See `DATA_SCHEMA.md`'s "Sort order (in `filterChords()`)" section for the full `tier()`/`categoryRank()`/`complexityScore()` logic and rationale.

## UI Features

### Filters (3-column `.controls` layout)
Key, Root Note, Quality, Complexity, Chord Type, Difficulty, Voicing (Primary/Other) — each a `<select>` wired to `filterChords()` via a `change` listener. **When adding a new filter dropdown OR checkbox that affects live rendering, remember to add its `addEventListener('change', filterChords)` line** — this was missed for Difficulty once, and separately for the `showKeys` checkbox too (both silently did nothing until wired up). See `DATA_SCHEMA.md` for the full current sort-order/filter behavior (overhauled substantially this session).

Results grid: `.results { grid-template-columns: repeat(4, 1fr); }` — a fixed 4-column layout (not `auto-fill`/`minmax` like earlier), so cards always lay out 4 across on desktop regardless of container width. Below the 768px mobile breakpoint this collapses to `1fr` (single column).

### Copy Options (checkboxes)
Include Description in Copy, Include Notes in Copy, Show Note Names in Diagram, Show Keys Chord Fits Into, Show Difficulty.

### Audio Playback
`playChord(frets)` — Web Audio API, triangle oscillators per string, staggered strum timing, gain envelope, ~1.6s ring. No external API/service, nothing to download or configure.

### Diagram Elements (SVG, via `drawChordDiagram()`)
- Open circles — open strings above the nut
- Fretted dots (black fill) — fretted notes, optionally labeled with note name
- Muted X — unplayed strings
- Barre line (thick stroke) — spans lowest→highest fretted string at the barre fret
- Fret numbers on the left (`viewBox` has a negative left margin so 2-digit fret numbers don't clip). `leftMargin = 10` (was 20) — narrowed so the gap from the badge edge to the fret numbers (rendered at `x=12`, `text-anchor=end`, ~13-14px wide for 2 digits) roughly matches the `rightPadding` (20) on the opposite side of the diagram, instead of leaving a lopsided extra-wide left margin.
- String names top and bottom
- Second line under the chord name shows the shape/position label

The `.chord-diagram` SVG itself is explicitly centered in its card via `display: block; margin: 0 auto;` (previously relied on default inline layout, which wasn't reliably centered).

### Card Layout — Play button & info text
The Play button (wrapped in a `playBtnWrap` div) and the notes/intervals/description/keys info block (`infoDiv`) both span the **full card width** with `width: 100%; box-sizing: border-box; padding: 0 10px; text-align: center`. This was previously narrower/left-shifted (a badge-formatting bug affecting the PNG export too). A more precise version was briefly attempted — a `transform: translateX(%)` shift computed to align these lines to the exact string-layout midpoint (accounting for the diagram's asymmetric left/right margins) — but once `leftMargin` was narrowed to 10 (making the diagram's margins roughly symmetric), the precise shift became unnecessary and was deliberately reverted back to plain full-card centering. Keep it simple like this unless the margins become asymmetric again.

### PNG Export
Canvas-rendered, mirrors the on-screen info (name, notes/intervals, description, keys per the Copy Options toggles). `centerX` for text is plain `canvas.width / 2` (a viewBox-aware centering calc was tried and reverted for the same reason as above).

## Style Decisions
- Black & white only, no grayscale
- SVG diagram text: chord name bold 16px, shape label 12px, string/fret labels 12px, note-in-dot labels 9px white
- PNG export text: bold 18px Arial, 22px line spacing, #333

## Testing
No automated test suite — this is a single-file app tested by hand in-browser. After any data-pipeline edit:
1. `node --check` the extracted `<script>` contents
2. Spot-check a handful of specific chord entries
3. Report before/after counts to the user (see `PIPELINE.md`)
4. Open in a browser and exercise the changed filter/feature directly — type checks don't verify UI behavior (this is how the missing `difficultySelect` event listener bug was caught: it looked correct in code but silently did nothing in the browser).

## Known History / Gotchas
See `PIPELINE.md` for the extract/mutate/serialize pattern and the specific bugs it was hardened against (self-matching dedup, tone-loss on auto-fix mutes).
