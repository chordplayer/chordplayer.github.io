# ChordPro Tags Reference

Reference for ChordPro directives: which are standard (per chordpro.org), which
`songsheet.html` actually parses, and Chorderoy's own extensions.

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
| `{start_of_tab:}` | `{sot:}` | Begin tab block |
| `{end_of_tab:}` | `{eot:}` | End tab block |

## Chorderoy extensions

| Directive | Purpose |
|---|---|
| `{roy_m:D}` | Chord ring-duration tag. Placed inline immediately before a chord bracket (e.g. `{roy_m: 1}[G]`). `D` is a duration in measures, or a fraction thereof (`.25`/`.5`/`.75`/etc). Carries duration only — never a position; a chord's position is always computed as the running sum of every duration before it in the song, so editing one chord's duration reflows every chord after it. Omitting the tag falls back to the "Auto chord duration" setting. `D` of `0` is a special "carry-over marker" (chord still ringing from earlier, not a new strum): no tick mark, excluded from the chord-playback schedule, but its label still displays. See `chordpro_time_tags.md` for the full spec. Not part of the ChordPro standard — namespaced with `roy_` to avoid colliding with any future standard tag. |

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
| `{define:}` | Custom chord fingering definition |
| `{chord:}` | Chord diagram directive |
| `{textfont:}`, `{chordfont:}`, `{textsize:}`, `{chordsize:}` | Font/size overrides |
| `{new_song:}` / `{ns:}` | Start a new song within one file |
| `{new_page:}` / `{np:}` | Force a page break |
| `{grid:}` | Inline grid marker |
| `{image:}` | Embed an image |

## Notes

- ChordPro has no native concept of measures/beats — `{roy_m:}` was invented for
  Chorderoy's playback/timing feature because no standard tag covers it.
- If a new Chorderoy-specific tag is ever added, prefix it `roy_` and document it here.
