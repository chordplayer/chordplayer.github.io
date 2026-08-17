# Backlog

Running list of work to pick up across sessions. Add/remove items as priorities change.

## Open

1. **Playback chord bug on reposition** — if the user manually repositions/scrolls the text in `#sheetPane` (moves it up), chord playback plays the chord it thinks is correct for that vertical position *plus* previous chords too. Needs investigation into the chord-scheduling/highlight logic (`chordSchedule`, `chordHighlightList`, `findChordAtOrBeforeMeasure`) to figure out why stale/previous chords are firing alongside the current one after a manual reposition.

2. **"Clear" control** — need a way to explicitly clear the currently loaded song so a fresh paste doesn't read as "editing the song that was last loaded/saved." Loading a new file already replaces state cleanly via `loadNewSource`; pasting into the Raw textarea while a song is already loaded needs an explicit clear path first.

3. **Adsense** — add to site.

4. **Landing page** — build one.

5. **Song Sheets page UI cleanup** — reclaim wasted vertical space and reorganize the controls row(s) on `songsheet.html` for a better overall UI experience. (Related recent work: Format/View toggle + diagrams checkbox row, `#chordproInput`/`#sheetPane` consolidation — this item is about tightening up what's there now, not redoing it.)

## Done (moved here for history, delete freely once stale)

- Save feature (source `.pro` + `.chordplayer`, directory picker, Safari download fallback)
- `.chordplayer` extension (was `.cpl`, blocked by Chrome Safe Browsing)
- Print/PDF top-of-page-1 gap (`#outputArea` not hidden for print)
- Raw/Song Sheet + Chordpro/Chordplayer view toggles, print diagrams checkbox
- Scroll jump fix + lead-in tuning
