# Chorderoy: ChordPro Measure Tagging for Teaching

## Purpose
Web app to add measure and beat-level timing data to ChordPro files for use in teaching/playalong scenarios. Enables precise sync between lyrics, chords, and musical timing.

## Core Functionality

### Input: ChordPro Files
- Parse standard ChordPro format (lyrics + inline chords)
- Extract metadata: title, artist, key, existing timing tags

### Auto-Estimation (Fixed Chord Duration)
The syllable-based estimator described in earlier drafts of this doc was tried and abandoned — it guessed badly often enough to be worse than a simple fixed rule. The current approach:
- The user picks a single **chord duration** for the whole song: whole measure, half measure, or quarter measure (UI control in the player bar, default half measure).
- Every chord marking with no explicit `{roy_m:}` tag is placed exactly one chord-duration after the previous chord *in the whole song* — a line break does **not** reset or snap this running position. A chord's ringing duration can carry across a line break, so the next line's opening words can land while the previous line's chord is still sounding, instead of every line being forced to start on a fresh measure.
- A chordless line gets a flat 1-measure filler span (so it still takes time to scroll past) without snapping the running position to any boundary.
- No syllable counting, lyric-shape heuristics, re-attack detection, or chord-only-line pattern borrowing — every chord marking (tagged or not) is assumed to occupy the same fixed duration. It won't be perfect, but it's predictable and closer than syllable-based guessing.
- Default metadata if missing:
  - `{time:4/4}` (if absent)
  - `{tempo:100}` (if absent)
  - `{capo:0}` (if absent)

### Measure Tagging System
- **Tags carry duration only, never position.** `{roy_m:D}` marks the ring duration (in measures, or a fraction — `.25`/`.5`/`.75`/etc.) of the chord it's placed immediately before, e.g. `{roy_m: 1}[G]` = "G rings for one full measure." There is no way to tag an absolute measure position directly.
- A chord's actual position (used for tick marks / grid lines in the UI) is always *computed*, never stored: it's the running sum of every duration before it in the whole song, continuous across line breaks.
- Chords using the fixed chord-duration default: no extra tag needed.
- Chords ringing for something other than the default: prepend `{roy_m:D}`, placed inline immediately before the chord bracket on the same line (`{roy_m: 1}[G] lyrics...`) so the tagged file keeps the same line count as the source.

### Smart Processing
- Any chord with an explicit `{roy_m:D}` tag uses that duration for how far the running cursor advances past it.
- Every other chord uses the fixed chord-duration default for that same advance.
- Because position is always derived from the cumulative sum, changing one chord's duration automatically reflows the position of every chord after it — no separate "reset" step needed.
- **`{roy_m:0}` is a carry-over marker, not a real strum**: it doesn't advance the cursor (the next chord starts at the same position), gets no tick mark of its own, and is excluded from the chord-playback audio schedule — but its label still displays on the sheet, to show a chord ringing from a previous line is still in effect.

### Editing model
The app keeps two independent in-memory copies once a song is loaded: the original source (read-only) and a "Chorderoy" tagged working copy (editable), selected via a toggle. Edits — whether typed directly into the tagged text, or made by clicking a chord in the rendered sheet (see below) — only ever touch the tagged copy; the original source is never mutated. Scroll speed and chord-playback timing are always driven by the tagged copy, regardless of which one is currently displayed.

### Interactive editing (implemented)
- Clicking a chord's label in the rendered sheet (with "Mark time" on, viewing "Chorderoy") pops up a small control: an editable chord-name field (retags the `[X]` bracket at that exact position) and a −/+ stepper for that chord's `{roy_m:D}` duration, in 0.25-measure steps floored at 0.
- Changing the "Auto chord duration" default re-tags the whole song from the original source at the new default (confirms first if there are unsaved edits).
- **Saving the tagged copy back out to a file is still not implemented** — edits persist in memory for the session only.

### Output
Extended ChordPro with all tags filled in:
```
{title:Song Name}
{artist:Artist}
{key:G}
{time:4/4}
{tempo:100}
{capo:0}

[G] Lyrics here
{roy_m: 0.5}[D] more lyrics
[Em] etc.
```

## Technical Approach
- Client-side processing (no server needed)
- Fixed chord-duration placement (see Auto-Estimation above)
- ChordPro parser
- Visual rendering with hierarchical line weights (measure vs. quarter)
- File load/save (local)

## Use Case
Teaching/playalong tools where students need to see exactly when chords change, at beat-level precision, synced to lyrics.
