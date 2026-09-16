# Future idea: rule-driven dynamic chord voicing generator

## The problem

`chordData` (in `chord-data.js`) is a hand-curated database of 614 voicings
covering 355 unique chord names (see `DATA_SCHEMA.md`). It's good - each
voicing has a verified idiomatic fingering, a computed difficulty rating,
and correct enharmonic spelling - but it's finite. A chord name that isn't
in it gets no diagram at all, unless the file itself supplies one via
`{define:}` (session-only, added this session - see `CHORDPRO_TAGS.md`).

The question this explores: instead of (or alongside) growing the curated
database by hand forever, could the app generate a reasonable voicing for
*any* chord name, on the fly, purely from its root + quality/intervals -
no fret data given, unlike `{define:}`?

## Feasibility

**Speed is a non-issue.** For each of the 6 strings, only frets whose note
matches one of the chord's required tones are candidates - typically 2-4
frets per string within any given ~5-fret window, not all ~15-20. A
constrained search within a sliding fret window is microseconds per chord,
and a typical loaded song only has 5-15 unique chord names. Generating
every chord in a freshly-loaded file, synchronously, would be unnoticeable.

**Quality is the real question**, and it's more tractable than it first
sounds. The hard part isn't finding *a* valid fingering (any note-correct
combination of frets works structurally) - it's finding the one a real
guitarist would actually play. But a surprising amount of that judgment is
already formalized in `DATA_SCHEMA.md`'s difficulty-classification rules,
which can double as **generation filters**, not just post-hoc scoring:

- Max fret span (frets touched, inclusive) ≤ 4
- No "sandwiched" muted string - a muted string with a played string
  (open or fretted) on both sides
- No more than ~4 simultaneously fretted notes (barre chords get a pass,
  since one finger covers several strings)
- Root note in the bass, unless the requested name explicitly specifies a
  slash bass

## Sketch of the algorithm

1. **Parse the chord name** into root + interval set (reuse/extend the
   existing enharmonic-spelling convention: every scale degree gets its
   own letter, never reuse a letter for two different degrees - already a
   documented, deterministic rule, not a judgment call).
2. **Generate candidates**: for a sliding base-fret window (start at fret
   0/open position, then try increasing base-frets), enumerate every
   fret-per-string combination (including muted) whose sounded notes
   exactly match the required chord tones (extra doublings of an existing
   tone are fine; missing or wrong tones are not).
3. **Filter** every candidate through the playability rules above -
   this alone should reject the large majority of technically-valid-but-
   nobody-plays-it-that-way shapes.
4. **Rank the survivors** to pick the one "default" voicing: prefer lower
   fret position, more open strings, fewer total fretted fingers. This is
   a scoring function, not a judgment call, so it's just as programmable
   as the filters.
5. **Compute difficulty** using the exact same rule set `DATA_SCHEMA.md`
   already documents for curated voicings, so generated and curated
   entries are rated consistently.

## Where this still falls short of hand curation

- **Fuzzy idiomatic-vs-not cases**: two shapes can both pass every filter
  rule while only one is the "real" one guitarists use - span/mute-
  sandwiching rules don't capture every finger-independence awkwardness.
  Likely irreducible without a much larger rule set, or acceptable as a
  rare miss.
- **Enharmonic-twin dedup across roots** (e.g. recognizing `C#maj7` and
  `Dbmaj7` as the same physical shape) took a real manual cleanup pass in
  the existing database (see `DATA_SCHEMA.md`'s "Duplicate voicing
  cleanup" section) - solvable algorithmically (compare fret-pattern
  arrays for exact matches) but is its own piece of work.
- A generator will sometimes need a small manual **override table** for
  the trickiest/most idiosyncratic chords, rather than trusting the
  algorithm end to end.

## Recommended scope, if pursued

Don't replace the curated database - keep it as the primary source (it's
already good and already built). Extend what `{define:}` support already
does this session: generate on the fly **only** for chord names that
aren't in `chordData` at all, injecting the result into
`chordData.standard._custom` the same way a `{define:}`-sourced voicing
already does (session-only, in memory, shows up in the legend
immediately). This gets verified quality for the common vocabulary and
infinite fallback coverage for anything exotic, without needing the
generator to be perfect - it only ever has to solve the hard "make this
look idiomatic" problem for names nobody's curated yet.

## Status

Not started - this is a design sketch from conversation, not a spec.
Next step, if picked up: prototype the candidate-generation + filter
pipeline against a batch of chords already in `chordData`, and compare
its output to the curated voicing to see how often they'd actually agree,
before deciding whether to build the full integration.
