# Editing Commands (songsheet.html)

## Keyboard shortcuts (either view, select lines first)

| Shortcut | Action |
|---|---|
| Ctrl+Alt+V | Wrap selected lines in a Verse section |
| Ctrl+Alt+C | Wrap selected lines in a Chorus section |
| Ctrl+Alt+B | Wrap selected lines in a Bridge section |
| Ctrl+Alt+T | Wrap selected lines in a Tab section |

No custom Cmd-key shortcuts exist outside what's listed below. Any other Cmd-key behavior (undo, copy/paste in text fields, etc.) comes from the browser's native text-field handling, not app code.

## ChordPlayer sheet view (tagged, with timing)

| Action | Effect |
|---|---|
| Click a chord label | Opens the duration/name popup for that chord |
| Shift-click the tick-marks row (not the chord name) | Inserts a new chord at that position and opens the popup for it |
| Drag a chord label left/right | Nudges that chord's timing |
| Right-click a chord | Marks it as the Scroll resume point (right-click again to clear) |

Duration popup (opened by clicking a chord):

| Control | Effect |
|---|---|
| − / + buttons | Step the chord's duration down/up |
| Enter (in the chord name field) | Commits the name and blurs the field |
| Delete button (×) | Deletes the chord |

## Chordpro sheet view (plain [chord] positions, no duration)

| Action | Effect |
|---|---|
| Click a chord | Opens the popup to rename it |
| Shift-click the lyric text | Inserts a new chord at that exact character; pastes the copied chord's name if one's been copied (see below), otherwise defaults to "C" |
| Drag a chord left/right | Moves it to a different character position on the same line |
| Hover a chord (no click needed) + Cmd/Ctrl+C | Copies that chord's name into an in-app clipboard |

Popup here has no −/+ duration stepper (plain ChordPro has no duration) — just the name field, and a chord picker (root A–G buttons, ♯/♭ toggle, Maj/min toggle, and a dropdown of matching chord names grouped by category) that rewrites the name field live as you click it. Emptying the name (instead of leaving a blank chord) deletes it outright.

**Copy/paste one chord:** hover any chord and press Cmd/Ctrl+C to copy its name (no need to click first), then Shift-click anywhere — including a different line — to paste a new chord with that name.

**Copy/paste one or more lines' chords:** select (highlight) one or more lines' lyric text, then **Ctrl+Cmd+C** (both modifiers together) to copy each line's chord progression separately. Select just the line where you want it to start and **Ctrl+Cmd+V** to paste — the whole copied set lands there and down (copied line 1 → the selected line, line 2 → the next line, etc.), *replacing* each target line's existing chords, positioned at the equivalent word (not exact character) on its own line. Stops early if the song runs out of lines. You'll likely need to drag individual chords afterward to fine-tune.
