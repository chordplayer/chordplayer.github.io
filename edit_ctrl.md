# Editing Keyboard Shortcuts (songsheet.html)

Select one or more full lines in the sheet, then use:

| Shortcut | Action |
|---|---|
| Ctrl+Alt+V | Wrap selected lines in a Verse section |
| Ctrl+Alt+C | Wrap selected lines in a Chorus section |
| Ctrl+Alt+B | Wrap selected lines in a Bridge section |
| Ctrl+Alt+T | Wrap selected lines in a Tab section |

No custom Cmd-key (metaKey) shortcuts are defined in the app. Any other Cmd-key behavior (undo, copy/paste, etc.) comes from the browser's native text-field handling, not app code.

## Mouse/pointer interactions (Tagged source view only)

These aren't keyboard shortcuts, but they're the rest of the in-sheet chord editing:

| Action | Effect |
|---|---|
| Click a chord label | Opens the duration/name popup for that chord |
| Shift-click the tick-marks row (not the chord name) | Inserts a new chord at that position and opens the popup for it |
| Drag a chord label left/right | Nudges that chord's timing |
| Right-click a chord | Marks it as the Scroll resume point (right-click again to clear) |

## Duration popup controls

Opened by clicking a chord (see above):

| Control | Effect |
|---|---|
| − / + buttons | Step the chord's duration down/up |
| Enter (in the chord name field) | Commits the name and blurs the field |
| Delete button | Deletes the chord |

