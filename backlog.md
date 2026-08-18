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
 
 

## Done (moved here for history, delete freely once stale)

- Strum Patterns for Playback — "Strum pattern" dropdown next to Auto chord duration in `songsheet.html`'s player bar: Single Chord Strum (default, original one-strum-per-chord behavior), Quarter Note Strum, Alternate Strum, Basic Strumming, Best Strum. Each non-default pattern drives an 8th-note grid (`1 + 2 + 3 + 4 +`) in `metronomeTick`, striking the currently-held chord down or up per slot; up-strums use only the highest 2-3 non-muted strings of that chord's voicing (`playChord` in `chord-data.js` gained a `direction` param). 4/4 only; other time signatures fall back to the original single-strum behavior. No new ChordPro tags — pattern is a player-only setting, not saved per-song.
- Save feature (source `.pro` + `.chordplayer`, directory picker, Safari download fallback)
- `.chordplayer` extension (was `.cpl`, blocked by Chrome Safe Browsing)
- Print/PDF top-of-page-1 gap (`#outputArea` not hidden for print)
- Raw/Song Sheet + Chordpro/Chordplayer view toggles, print diagrams checkbox
- Scroll jump fix + lead-in tuning
- Playback chord bug on reposition (was: chord playback firing stale/previous chords alongside the current one after a manual reposition) — root cause was a beat-alignment bug: the sustain re-strum fired on a tick-counted `beatInMeasure === 1`, which only lines up with a real measure boundary when resuming from a whole measure; a mid-song resume from a fractional position sent false "beat 1"s that don't correspond to real boundaries, re-striking the current chord. Fixed by resuming on a snapped, true chord/measure boundary and detecting real measure-boundary crossings instead of the resume-relative tick count.
