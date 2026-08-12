# Guitar Chord Diagram Generator

A web-based interactive chord diagram tool for guitarists. Browse 352 unique chords (612 voicings total), view accurate diagrams with music theory notation, play chord audio, and export as PNG images.

## Features

🎸 **352 Chords, 612 Voicings**
- Standard tuning
- Interval-accurate note spelling (letter-collision-free — e.g. E# not F for a raised 5th)
- Multiple voicings per chord, with one marked as the "primary" (canonical) voicing
- Open, barre, fretted, power, and "High Triad" (top-3-string) shapes

🔊 **Audio Playback**
- "Play" button synthesizes the chord via the Web Audio API (staggered strum, no external services/downloads)

📋 **Filters**
- Key (18 major/minor keys), Root Note, Quality (Major/Minor — neutral chords like power chords match either), Complexity (Triads/Power/Complex), Chord Type (Open/Barre/Fretted/Power/High Triads), Difficulty (additive: Easy / Easy/Medium / Easy/Medium/Hard — defaults to Easy/Medium), Voicing (Primary/Other)

📸 **PNG Export**
- Copy button copies diagram + metadata to clipboard
- Optional: note names in diagram, description, notes, and compatible keys

## Quick Start

1. **Open** `chord-diagrams.html` in your web browser
2. **Filter** by key, root, quality, complexity, type, difficulty, and/or voicing
3. **View** the diagram, hit **Play** to hear it
4. **Copy** the PNG to your clipboard and paste anywhere

## Reading the Diagram

```
    ×           (muted string)
    ○           (open string)
  |●|           (fretted note, optionally labeled with letter name)
    1 2 3 4 5   (fret numbers on left)

E A D G B e     (string names bottom)
```

Each card shows: chord name, shape label (e.g. "Open", "E-shape barre", "5th position"), the diagram, a Play button, notes + intervals, description, and (optionally) compatible keys.

### Keys display
Shows two tiers: keys the chord is **fully** diatonic to, and — after a `/` — keys where the core tones (root/2nd/3rd/4th/5th) fit but an extension (6/7/9/11/13) doesn't. This means altered-bass and extended chords still show up under their own root's key even when one color tone isn't in the literal 7-note scale.

### Difficulty
Rated easy/medium/hard based on chord type, fret span, and finger count — not just span. See `DATA_SCHEMA.md` for the exact rule.

## What's Included

- **chord-diagrams.html** — the complete single-file application (no build step, no dependencies)
- **README.md** — this file (user guide)
- **DEVELOPMENT.md** — architecture and UI overview
- **DATA_SCHEMA.md** — the chord data format, field-by-field
- **PIPELINE.md** — how bulk data edits are made (Node.js extract/mutate/serialize pattern)
- **CLAUDE.md** — working agreement for AI-assisted development on this project

## Browser Compatibility
Chrome/Chromium, Firefox, Safari, Edge. Works offline after initial load.
