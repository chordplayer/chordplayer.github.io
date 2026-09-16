# ChordPro Tags Reference

Reference for ChordPro directives: which are standard (per chordpro.org), which
`songsheet.html` actually parses, and ChordPlayer's own extensions.

## Implemented in `songsheet.html`

Standard directives handled by `parseChordPro()`:

| Directive | Aliases | Purpose |
|---|---|---|
| `{title:}` | `{t:}` | Song title |
| `{subtitle:}` | `{st:}` | Subtitle |
| `{artist:}` | | Artist name |
| `{key:}` | | Song key |
| `{capo:}` | | Capo position |
| `{time:}` | | Time signature (e.g. `4/4`) |
| `{tempo:}` | | Tempo in BPM |
| `{comment:}` | `{c:}`, `{comment_italic:}`, `{ci:}` | Inline comment line |
| `{start_of_chorus:}` | `{soc:}` | Begin chorus section |
| `{end_of_chorus:}` | `{eoc:}` | End chorus section |
| `{start_of_verse:}` | `{sov:}` | Begin verse section |
| `{end_of_verse:}` | `{eov:}` | End verse section |
| `{start_of_bridge:}` | `{sob:}` | Begin bridge section |
| `{end_of_bridge:}` | `{eob:}` | End bridge section |
| `{start_of_tab:}` | `{sot:}` | Begin tab block - per-line, not per-block: any line inside the block that contains a real `[chord]` bracket is parsed exactly like a normal chorded line (chords, `{x_cpl_m:}` timing, legend/playback participation, grid rendering) instead of being taken verbatim; a line with no brackets is unaffected and still rendered as-is. Real ASCII fretboard notation never contains `[`/`]`, so this only ever activates on a line that actually has bracket syntax in it. |
| `{end_of_tab:}` | `{eot:}` | End tab block |
| `{define:}` | | Custom chord fingering definition (`NAME base-fret N frets F1 F2 F3 F4 F5 F6`, standard 6-string ChordPro syntax). Only affects `chordData` when `NAME` isn't already a known chord - see "ChordPlayer extensions" below; a name chordData already has a diagram for is left alone. |

## ChordPlayer extensions

Not part of the ChordPro standard. Per the spec's own custom-extension
convention, any directive named `x_...` must be silently ignored by an
application that doesn't handle it — no warning, no error — so any
ChordPlayer file stays a perfectly ordinary, readable chord chart in any
other spec-compliant ChordPro app, even one with zero ChordPlayer support.
`cpl` (ChordPlayer) is this app's own namespace within that convention, so
every ChordPlayer-specific tag is `x_cpl_<name>`.

| Directive | Purpose |
|---|---|
| `{x_cpl_m:D}` | Chord ring-duration tag. Placed inline immediately before a chord bracket (e.g. `{x_cpl_m: 1}[G]`). `D` is a duration in measures, or a fraction thereof (`.25`/`.5`/`.75`/etc). Carries duration only — never a position; a chord's position is always computed as the running sum of every duration before it in the song, so editing one chord's duration reflows every chord after it. Omitting the tag falls back to the "Auto chord duration" setting. `D` of `0` is a special "carry-over marker" (chord still ringing from earlier, not a new strum): no tick mark, excluded from the chord-playback schedule, but its label still displays. See `chordpro_time_tags.md` for the full spec. |
| `{x_cpl_v:URL}` | Reference video tag. A header directive (like `{tempo:}`/`{time:}`, one per song, not positional) holding a YouTube URL — shown/editable in the "Reference Video" panel on `songsheet.html` for manually matching tempo by ear (start the video, tap `#tapTempoBtn` to the beat, then hit Scroll) rather than anything programmatically synced to playback. Auto-fills the field on load when present; written back on Save from whatever's currently in the field, and removed entirely if the field is cleared. If no `{x_cpl_v:}` tag is present, the field falls back to scanning the song's lyrics/comments for a bare YouTube link, for songs saved before this tag existed. |

**Legacy naming**: earlier ChordPlayer files used `{cpl_m:}`/`{cpl_v:}`,
without the `x_` prefix - not spec-compliant (a strict reader has no
obligation to silently ignore an unprefixed unknown directive the way it
does for `x_`-prefixed ones). `parseChordPro()` still accepts the old
names on read, so nothing written before this rename breaks, but
`buildTaggedChordPro()`/Save always write the current `x_cpl_*` names -
a file gets migrated to the new names the moment anything in it is
edited (any edit re-serializes the whole song), not merely by loading
or re-saving it untouched.

`{define:}` itself is standard ChordPro (see the table above), but what
`songsheet.html` does with it is ChordPlayer-specific: on load, any
`{define:}` name not already in `chordData` is computed into a session-only
voicing (frets, notes, intervals, quality) and added to
`chordData.standard._custom`, purely in memory - it shows up as a real
diagram in the legend for that browser session, but is never written back
to the file and doesn't survive a page reload.

## Standard ChordPro directives not yet implemented

These exist in the ChordPro spec but `songsheet.html` doesn't parse them (unrecognized
directives are currently silently ignored by `parseChordPro()`):

| Directive | Purpose |
|---|---|
| `{composer:}` | Composer name |
| `{lyricist:}` | Lyricist name |
| `{album:}` | Album name |
| `{year:}` | Release year |
| `{duration:}` | Track duration |
| `{comment_box:}` / `{cb:}` | Boxed comment |
| `{highlight:}` | Highlighted comment |
| `{start_of_grid:}` / `{sog:}`, `{end_of_grid:}` / `{eog:}` | Chord grid block |
| `{chord:}` | Chord diagram directive |
| `{textfont:}`, `{chordfont:}`, `{textsize:}`, `{chordsize:}` | Font/size overrides |
| `{new_song:}` / `{ns:}` | Start a new song within one file |
| `{new_page:}` / `{np:}` | Force a page break |
| `{grid:}` | Inline grid marker |
| `{image:}` | Embed an image |

## Notes

- ChordPro has no native concept of measures/beats — `{x_cpl_m:}` was invented for
  ChordPlayer's playback/timing feature because no standard tag covers it.
- If a new ChordPlayer-specific tag is ever added, name it `x_cpl_<name>` and document it here.
