# Future idea: companion browser extension for YouTube pitch-shifting

## The problem

ChordPlayer's YouTube reference video (`{x_cpl_v:}` tag / Video panel) can be
slowed down or sped up via the YouTube IFrame API's `setPlaybackRate()`,
which already preserves pitch (no chipmunk effect) - that's YouTube's own
built-in time-stretching, not something this app does.

What YouTube's API does *not* expose is independent **pitch** shifting -
there's no way to play a reference video in a different key while keeping
its speed the same. This isn't a gap in ChordPlayer's implementation; it's
a hard limitation of the embedded IFrame API, and the audio itself is
cross-origin sandboxed, so no page-level JS (Web Audio or otherwise) can
intercept or reprocess it. See the [DEVELOPMENT.md](DEVELOPMENT.md) /
session discussion this file was split out of for the full reasoning.

The one thing ChordPlayer *can* already do is transpose the chord sheet
itself (see the Transpose control in `songsheet.html`) and its own
synthesized "Play chords" audio - just not the YouTube reference track.

## The idea

A separate, standalone browser extension (not part of this repo) that runs
directly on `youtube.com` (not inside a sandboxed iframe, so it has real
access to the page's `<video>` element) could reroute that video's audio
through the Web Audio API and apply real-time pitch shifting independent
of playback speed - exactly what practice tools like Amazing Slow Downer
or Moises do, but live against any YouTube video.

If ChordPlayer's Transpose control also broadcast a `window.postMessage`
event on transpose, that extension's content script could listen for it
and shift the reference video's pitch to match - "transpose the chart"
and "transpose what you hear" moving together.

## Technical sketch

1. **Content script on youtube.com** grabs `document.querySelector('video')`
   and reroutes its audio: `audioContext.createMediaElementSource(videoEl)`
   → a pitch-shift node → `audioContext.destination`. This disconnects
   YouTube's default output and pipes it through Web Audio instead - a
   well-documented pattern, not novel.
2. **Pitch-shift DSP** - the one genuinely hard part, but already solved:
   [soundtouchjs](https://github.com/cutterbl/SoundTouchJS) (a JS port of
   the SoundTouch library used by several commercial slow-downer apps)
   does real-time independent pitch/tempo shifting via an
   `AudioWorkletNode`. Use it, don't reimplement it.
3. **UI** - a small injected slider/±control, or driven entirely by
   ChordPlayer's own Transpose buttons via `postMessage` (with a handshake
   ping so ChordPlayer can detect whether the extension is installed and
   only show the "sync pitch" option when it is).
4. **Distribution**:
   - *Firefox*: self-distributable - Mozilla will sign a `.xpi` for free
     (automated AMO signing, no store listing/review required), and it can
     install with one click and auto-update via an `update_url` pointing
     at GitHub Releases.
   - *Chrome*: since ~2021, Chrome blocks installing unpacked/sideloaded
     extensions for regular users outside Developer Mode - a GitHub-only
     release means real friction for non-technical Chrome users. Publishing
     to the Chrome Web Store (one-time review, small one-time developer
     fee) removes that friction and is usually less total effort than
     maintaining a sideload workflow.

## Effort estimate

A working MVP (audio reroute + soundtouchjs + a manual slider, sideloaded
for personal use) is roughly a weekend project for someone comfortable
with JS - the hard DSP problem is already solved by an existing library,
so this is mostly integration work. The ChordPlayer-side `postMessage`
hook would be a small, separate addition on top once the extension exists.

## Status

Not started. This would be its own standalone project/repo, not a change
to `chord-diagrams.html` or `songsheet.html` - see backlog.md.
