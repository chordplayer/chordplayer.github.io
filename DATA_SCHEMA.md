# Chord Data Schema

`chordData.standard[chordKey]` is an array of **voicings**. Each voicing is a 15-element array:

| Index | Field | Type | Notes |
|---|---|---|---|
| 0 | name | string | e.g. `"Am7"`, `"A/C#"`. May end in `*` (legacy flag marker — unused now, no entries currently flagged) |
| 1 | frets | array[6] | low-E to high-e. `'x'` = muted, `0` = open, number = fret |
| 2 | notes | array of strings | interval-correct spelling — never reuse a letter for two different scale degrees within one chord |
| 3 | intervals | array of strings, parallel to `notes` | e.g. `"1"`, `"b3"`, `"#5"`, `"b7"`, `"9"`, `"11"`, `"13"` |
| 4 | description | string | e.g. `"major triad"`, `"diminished triad,diminished 7th"` |
| 5 | root | string | letter + optional accidental |
| 6 | type | string | `'open'` \| `'barre'` \| `'fretted'` \| `'power'` \| `'High Triads'` |
| 7 | modifiers | array | legacy field (e.g. `['diminished']`) — UI checkbox for it was removed, field retained |
| 8 | barrePosition | number \| null | |
| 9 | positionLabel | string \| null | e.g. `"Open"`, `"5th position"`, `"E-shape barre"`, `"A-shape barre"`, `"D-shape barre"`, `"Other barre"`, `"Fretted"`, `"Power-E"`, `"Power-A"`, `"High Triad"` |
| 10 | flagged | boolean | legacy — always `false` now, flagging system fully resolved/removed from UI |
| 11 | reviewNote | string \| null | legacy — always `null` now |
| 12 | quality | string | `'major'` \| `'minor'` \| `'neutral'` (power/sus-type chords compatible with either) |
| 13 | primary | boolean | exactly one `true` per unique chord name (the canonical voicing) |
| 14 | difficulty | string | `'easy'` \| `'medium'` \| `'hard'` — see rule below |

## Current dataset size (as of last edit)
- 612 total voicings, 352 unique chord names
- By type: open 272, barre 123, fretted 67, High Triads 82, power 68
- By quality: major 273, minor 194, neutral 145
- By difficulty: easy 171, medium 365, hard 76

(These counts dropped substantially from an earlier 1,164/711 baseline after a duplicate-cleanup pass — see "Duplicate voicing cleanup" below. Difficulty counts also shifted since fewer `open`/`fretted` voicings remain overall.)

## Difficulty rule
Computed from `type` + fret span + finger count + fingering pattern, not purely subjective:
- `open` → **easy**, EXCEPT (checked in this order):
  1. spans 4+ frets touched → **hard** (16 voicings — e.g. `Am/C`: `[8,12,x,x,x,0]`)
  2. spans more than 2 frets touched AND the low-E string is fretted AND the next fretted string (skipping over any open/muted A and D strings) is on G, B, or e → **hard** (15 voicings — e.g. `G/F`: `[1,x,0,0,0,3]`. A stretch across muted/open middle strings, harder than the finger-count rule below even with only 2-3 fingers)
  3. frets more than 3 strings (4+ fingers needed) → **medium**
  4. has a "sandwiched" muted string — any muted string with a played string (open OR fretted) on both sides, i.e. you have to selectively deaden a string in the interior of the chord rather than just swiping across the low string(s) at the start → **medium** (128 voicings total match this pattern; 45 of them were newly reclassified from easy — e.g. `Em7`: `[0,x,0,0,0,0]`, where even a mute sandwiched between two open strings counts)
  5. otherwise → **easy**
- `power`, `barre`, `High Triads` → always **medium**
- `fretted` → depends on frets *touched* (`max fret - min fret + 1`, i.e. counting-inclusive, not a raw subtraction — e.g. frets 3,4,5 = 3 frets touched):
  - span ≥ 3 → **hard**
  - span = 2 **and** 4+ fretted strings (a same-fret note on two non-adjacent strings with a different fret on the string between them — an interrupted partial-barre pattern) → **hard**
  - otherwise → **medium**

This intentionally does NOT use raw fret-span alone — barre and power chords routinely span 2-3 frets but are easy/medium in practice, while some `fretted` voicings are hard despite a small span because of awkward finger-independence patterns (same fret on non-adjacent strings, muted string sandwiched between fretted ones).

## Enharmonic spelling convention
Every scale degree gets its own letter (A–G); never reuse a letter for two different degrees in the same chord. E.g. Aaug = A, C#, E# (not F) — the augmented 5th must be spelled as a raised letter-5 (E#), not reinterpreted as F.

## Key display
`getKeysForChord(notes)` — strict: every note must be in the 7-note scale.
`formatKeysDisplay(notes, intervals)` — two-tier: strict matches, then (if any) a `/ ...` suffix for keys where only the **core tones** (root/2nd/3rd/4th/5th) are diatonic but an extension (6/7/9/11/13, any accidental) is not. This exists because altered-bass / extended chords are normally still considered "in" their root's key even when a color tone isn't in the literal 7-note scale — without it, many altered-bass chords showed zero keys, even for their own obvious root key.
- If strict-only: plain list, no marker (e.g. `A, D, E, Bm, C#m, F#m`)
- If extended-only (strict is empty): prefixed with `/ ` (e.g. `/ Ab, Eb, Cm, Fm`)
- If both: `strict / extended`

Note: the **Key filter dropdown** still uses strict-only matching (`getKeysForChord`) — it was not changed to two-tier when the display was.

## Sort order (in `filterChords()`)
Overhauled in this session into a two-tier scheme, replacing the old flat "triad/power/everything else" ranking:

1. **Root** (chromatic order: naturals/sharps first, then flats — see `ROOT_ORDER` array)
2. **Tier**: `{major triad, minor triad, power}` = tier 0, everything else = tier 1
   - `isTriad(v)` now requires the intervals to be **exactly** `{1, 5, and one of 3/b3}` (a real plain triad) — not just "3 notes total, no modifiers." This was tightened this session after finding a 3-note-but-no-5th chord (`A6(no 5th)`, intervals `1,3,6`) was wrongly slipping into tier 0.
   - *(tier 0 only)* sub-rank: major triad (0) → power (1) → minor triad (2)
3. *(tier 1 only)* **Category** — the dominant key within tier 1, groups voicings by chord "family" so e.g. all sus2 chords sit together: 7th-family (0) → sus2-family (1) → sus4-family (2) → altered-bass-family (3) → catch-all (4). A voicing only qualifies for a named family if it has **exactly one** alteration matching that family (e.g. `Asus2add6`, which has two alterations, falls to catch-all rather than the sus2 family — this is deliberate, confirmed with the user).
4. *(tier 1 only)* **Complexity score** — orders voicings *within* the same category. Counts alterations from a plain major/minor triad: +1 for each interval outside `{1,3,b3,5}`, **+1 if the 3rd is missing** (sus chords), **+1 if the 5th is missing**, +1 if the name has an altered-bass `/`. Missing the 3rd/5th is weighted as costing more than adding a color tone on top of an intact triad, since losing the note that defines major-vs-minor (or the 5th) is a bigger structural change than just decorating a triad — this was a deliberate design choice, not an accident.
5. Quality (major/neutral before minor)
6. Name (alphabetical)
7. Primary voicing first
8. Type (alphabetical)

Difficulty is NOT part of the sort order — only available as a filter.

## UI filters (all in the 3-column `.controls` layout)
Key, Root Note, Quality (All/Major/Minor/Neutral — exact match against `voicing[12]`, EXCEPT `type === 'power'` chords always bypass this filter and show under any quality selection, since a power chord has no 3rd at all and so doesn't contradict major or minor; sus2/sus4 chords do NOT get this bypass — they're exact-match neutral like everything else), Complexity (Triads/Power/Complex), Chord Type (Open/Barre/Fretted/Power/High Triads), Difficulty (additive — Easy / Easy/Medium / Easy/Medium/Hard), Voicing (Primary/Other).

Difficulty filter is **additive/cumulative**, not exact-match: selecting a level shows that level and everything easier. Default on page load is "Easy/Medium". There is no blank "All" option — "Easy/Medium/Hard" (the hard tier) serves that purpose.

Controls are laid out in 3 columns: col 1 = Difficulty/Key/Root Note/Quality, col 2 = Voicing/Complexity/Chord Type, col 3 = Copy Options checkboxes. All columns top-aligned (`align-items: flex-start`).

## Difficulty display
The `(Easy)`/`(Medium)`/`(Hard)` label is rendered **inside the SVG itself** (in `drawChordDiagram()`), positioned above the chord name, sharing the exact same `x` coordinate and font as the sub-label below the name — this guarantees pixel-perfect centering in both the on-screen card and the PNG export (which just rasterizes the same SVG), regardless of the diagram's asymmetric `viewBox`. Controlled by the "Show Difficulty" checkbox (`showDifficulty`, default checked), passed into `drawChordDiagram(voicing, tuning, showNoteNames, showDifficulty)`. When shown, `drawChordDiagram` adds a `difficultyOffset` (20px) to `topPadding` and to the y-coordinates of the chord title and sub-label, so everything shifts down together — do not add a separate difficulty element outside the SVG (this was tried and reverted; centering broke because the SVG's own coordinate space doesn't match the outer card's).

## Duplicate voicing cleanup (large pass, one session)
Started at 1,164 voicings / 711 names, ended at 612 voicings / 352 names. "Duplicate" here specifically means two (or more) *differently-named* voicings sharing the **exact same `frets` array**. Cross-root sharing (the same shape legitimately being a different valid chord from a different root) was explicitly out of scope and is fine to keep — only within-root duplicates and clearly-inferior alternate names were removed. Rules applied, roughly in the order they were discovered:

1. **Bass-doubling rule**: for a duplicate pair where one name treats an "extra" tone as a real extension (e.g. `A7/G`) and the other calls it "altered bass" (`A/G`), check whether that tone is *doubled* elsewhere in the voicing (present in its normal upper-voice register) or *only* appears as the bass note. Doubled → the extension name is correct. Bass-only → the plain-triad/altered-bass name is correct (the extension name oversells a note that's really just a foreign tone stuck under the chord).
2. **Bass-mismatch rule**: if a slash name's stated bass (the letter after `/`) doesn't match the note that's actually lowest in the voicing, that name is simply wrong — drop it in favor of a name whose bass claim (or implicit root-position assumption, for non-slash names) is correct.
3. **Register rule** (`6` vs `add13`, `2` vs `add9`, etc.): compute the actual semitone distance from the lowest occurrence of the root to the lowest occurrence of the color tone. `< 12` semitones → simple label (`2`, `4`, `6`). `>= 12` → compound label (`9`, `11`, `13`). This is per-voicing, not per-chord-name — the same note can be simple in one fingering and compound in another.
4. **`dim` vs `m7(b5)` vs `o7`**: a `dim` (triad-only) name is wrong if a 4th note (a 7th) is actually present. Between `m7(b5)` and `o7`: `o7` specifically means a *fully* diminished 7th (bb7), so it's wrong whenever the actual interval is a b7, not bb7 — `m7(b5)` is correct for that case. This produced 3-way ties (`dim` + `m7(b5)` + `o7` all sharing one fretting) more than once.
5. **`sus2`/`sus4` when a 3rd is actually present**: sus chords replace the 3rd; if the interval list has a 3rd *and* a 2nd/4th, the sus name is wrong — prefer the plain triad name (major/minor + altered bass), which doesn't make a false claim even if it doesn't explicitly call out the extra tone.
6. **Genuine dual-suspension** (both 2nd and 4th present, no 3rd at all — a real edge case, not a naming bug): neither `sus2` nor `sus4` alone is complete. Resolved by picking whichever of the 2nd/4th sits at the **lower actual pitch** in that specific voicing — the lower one reads as more foundational, the higher one as a passing color tone.
7. **Malformed slash names**: a few names had an extension term (`add11`, `add9`) stuck in the bass-note slot (e.g. `B7/add11`) — a leftover bug from an earlier bulk edit, not a duplicate-naming issue. Fixed by renaming (stripping the bogus slash) rather than deleting, since these were often unique voicings, not true duplicates. One instance (`E7/add11`) *was* a true duplicate of a correctly-named `E7/A` and was removed instead.
8. **Half-step "avoid note" clashes**: found and removed voicings where two chord tones sound a genuine semitone apart in close register (not just pitch-class-adjacent an octave apart) — e.g. simultaneous b5/5, or a major 3rd against a natural 11th (the classic reason real players raise a natural 11 to `#11` or drop the 3rd). Two systemic **false positives** were identified and excluded from this rule: `maj7` chords (root vs. major-7 is always a semitone apart — that's the entire point of a maj7 sound) and `madd9` chords (b3 vs. 9 is always a semitone apart structurally, and it's a normal, pleasant, extremely common voicing). Only ~17 voicings were genuine "sounds bad" clashes after excluding those two patterns.
9. **Exotic-name pruning** (largest single cut, ~237 voicings across two passes): many duplicate groups had one clearly practical name (no altered-bass slash, no double-sharp/double-flat spelling) alongside several theoretically-valid-but-unusable alternates — almost always a symptom of augmented/diminished chord symmetry (an augmented triad or fully-diminished-7 chord can be legitimately "rooted" from any of its own notes). Rule: **prefer a non-slash (root-position) name over any slash name for the same shape.** When no non-slash name exists in a group, prefer least exotic spelling (fewest double-accidentals) with ties broken by which candidate's root sits at the lowest actual pitch in that voicing.
   - **Important refinement**: when applying the exotic/lowest-root tiebreak to groups of 3+ names, first cluster names by "same quality + enharmonic root" (e.g. `F#7sus4/Db` and `Gb7sus4/Db` are twins, not rivals) and score whole clusters, not individual names — otherwise a legitimate enharmonic twin pair gets wrongly collapsed to just one survivor.

After all of this, the remaining cross-root sharing (124 fret patterns with 2 names) is exclusively the intentional case: identical quality, enharmonic root spelling (e.g. `C#maj7` vs `Dbmaj7`). Zero within-root duplicates remain, and zero groups mix genuinely different chord qualities on one shape without a clear winner.

**Pipeline note**: this cleanup was done interactively in a Node REPL-style workflow (see `PIPELINE.md`), not as a single canonical script — rules were discovered and refined mid-session as counter-examples came up (e.g. the initial "any pitch-class-adjacent tones = bad" clash rule had to be narrowed after `maj7`/`madd9` false positives were caught). If extending this cleanup further, re-derive the rules from this document rather than assuming a rerunnable script exists.
