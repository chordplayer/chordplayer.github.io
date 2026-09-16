# Backlog

Running list of work to pick up across sessions. Add/remove items as priorities change.

## Open

1. **"Clear" control** — need a way to explicitly clear the currently loaded song so a fresh paste doesn't read as "editing the song that was last loaded/saved." Loading a new file already replaces state cleanly via `loadNewSource`; pasting into the Raw textarea while a song is already loaded needs an explicit clear path first.

2. **Adsense** — add to site.

3. **Landing page** — build one.

4. **Song Sheets page UI cleanup** — reclaim wasted vertical space and reorganize the controls row(s) on `songsheet.html` for a better overall UI experience. (Related recent work: Format/View toggle + diagrams checkbox row, `#chordproInput`/`#sheetPane` consolidation — this item is about tightening up what's there now, not redoing it.)

5. **Document Chordplayer extension as open source - the new tags, not my webapp**

6. **Help screen**

7. **Contact**

8. **Terms of Use**

9. **Companion browser extension for YouTube pitch-shifting** — standalone project (not a change to `chord-diagrams.html`/`songsheet.html`), lets the reference video be played back in a different key independent of speed, optionally synced to the Transpose control via `postMessage`. See [PITCH_SHIFT_EXTENSION_IDEA.md](PITCH_SHIFT_EXTENSION_IDEA.md) for the full writeup (not to be confused with item 5, which is about documenting ChordPlayer's own ChordPro tag extensions).

10. **Handle standard ChordPro `{start_of_grid:}`/`{end_of_grid:}` blocks** — low priority, grids don't seem widely used in the wild. Currently `parseChordPro()` doesn't recognize these directives at all (no `inGrid` tracking like `inTab` has), so a grid block's content line (e.g. `| G . C . | G . C . |`) has no `[...]` brackets and gets swallowed whole as literal lyric text - it renders as inert junk text, contributes no chords to the legend, and generates no `{x_cpl_m:}` timing at all (verified directly, not just inferred from the code). If ever implemented: would need to track grid-block state similar to `inTab`, parse each `|`-delimited bar into evenly-spaced slots (`.` = hold/no new strike, `%` = repeat previous bar, `*N` = repeat N times - see chordpro.org spec), and synthesize the equivalent `{x_cpl_m:D}[chord]` sequence inline so it plays/displays exactly like a normal chorded line.

11. **Handle standard ChordPro `{chord: NAME}` directive** — non-positional, forces `NAME` into the diagram legend without it needing to appear anywhere via `[NAME]` brackets in the lyrics (unlike `[chord]`, it doesn't affect what's actually played/timed - it's a "also show this diagram" hint, e.g. offering `Cadd9` as a suggested substitution/variation for a song that actually plays plain `[C]`, or just curating which diagrams appear independent of the song body). Currently unrecognized by `parseChordPro()`, silently dropped - no legend effect at all. If implemented: collect `{chord:}` names onto the parsed song (similar to `customDefines`) and fold them into `collectChordNames`/`currentChordNames` so they show up in the legend even with zero `[NAME]` occurrences in the body.

12. **Rule-driven dynamic chord voicing generator** — explored in conversation, not started. Generate a playable fretboard voicing for any chord *name* on the fly (no fret data given, unlike `{define:}`) by enumerating fret combinations matching the required chord tones, filtering through the same playability rules `DATA_SCHEMA.md` already documents for difficulty classification (max 4-fret span, no sandwiched mute, ≤4 fretting fingers, root in the bass), then ranking survivors (lower position, more open strings, fewer fingers) to pick a default. See [CHORD_VOICING_GENERATOR_IDEA.md](CHORD_VOICING_GENERATOR_IDEA.md) for the full sketch, including where it'd likely fall short of hand curation (fuzzy idiomatic-vs-not cases, enharmonic-twin dedup) and the recommended scope (fallback only for names not already in `chordData`, injected into `chordData.standard._custom` the same way `{define:}` already does - not a wholesale DB replacement).
 
 

## Done (moved here for history, delete freely once stale)

- Strum Patterns for Playback — "Strum pattern" dropdown next to Auto chord duration in `songsheet.html`'s player bar: Single Chord Strum (default, original one-strum-per-chord behavior), Quarter Note Strum, Alternate Strum, Basic Strumming, Best Strum. Each non-default pattern drives an 8th-note grid (`1 + 2 + 3 + 4 +`) in `metronomeTick`, striking the currently-held chord down or up per slot; up-strums use only the highest 2-3 non-muted strings of that chord's voicing (`playChord` in `chord-data.js` gained a `direction` param). 4/4 only; other time signatures fall back to the original single-strum behavior. No new ChordPro tags — pattern is a player-only setting, not saved per-song.
- Save feature (source `.pro` + `.chordplayer`, directory picker, Safari download fallback)
- `.chordplayer` extension (was `.cpl`, blocked by Chrome Safe Browsing)
- Print/PDF top-of-page-1 gap (`#outputArea` not hidden for print)
- Raw/Song Sheet + Chordpro/Chordplayer view toggles, print diagrams checkbox
- Scroll jump fix + lead-in tuning
- Playback chord bug on reposition (was: chord playback firing stale/previous chords alongside the current one after a manual reposition) — root cause was a beat-alignment bug: the sustain re-strum fired on a tick-counted `beatInMeasure === 1`, which only lines up with a real measure boundary when resuming from a whole measure; a mid-song resume from a fractional position sent false "beat 1"s that don't correspond to real boundaries, re-striking the current chord. Fixed by resuming on a snapped, true chord/measure boundary and detecting real measure-boundary crossings instead of the resume-relative tick count.
