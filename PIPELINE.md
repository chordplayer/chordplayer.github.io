# Bulk Data Edit Pipeline

`chord-diagrams.html` embeds `chordData` as a JS object literal inside a `<script>` tag. There's no build step, so bulk edits to the data (renaming chords, fixing spellings, adding fields, dedup, etc.) are done with a repeatable Node.js script, not hand-editing the HTML.

## Pattern

1. **Extract**: read the HTML, find `const chordData = {`, brace-match to find the matching closing `}`, slice out the object-literal text.
2. **Load**: write `module.exports = <objText>;` to a temp file, `require()` it (gives you a real JS object to mutate).
3. **Mutate**: loop over `Object.values(chordData.standard)` → voicings, make the change.
4. **Serialize**: a custom `serialize()` function (recursive, handles arrays/objects/primitives) turns the mutated object back into the same array-literal source format — do NOT use `JSON.stringify` directly on the whole object, it won't preserve the exact formatting conventions already in the file.
5. **Splice back**: `html.slice(0, start) + out + html.slice(blockEnd)`, write to `chord-diagrams.html`.
6. **Verify**: `node --check` on the re-extracted `<script>` contents to catch syntax errors before considering the edit done.

## Minimal template

```js
const fs = require('fs');
const HTML_PATH = '/path/to/chord-diagrams.html';
const html = fs.readFileSync(HTML_PATH, 'utf8');

const marker = 'const chordData = {';
const start = html.indexOf(marker);
const braceStart = html.indexOf('{', start);
let depth = 0, i = braceStart;
for (; i < html.length; i++) {
  if (html[i] === '{') depth++;
  else if (html[i] === '}') { depth--; if (depth === 0) { i++; break; } }
}
const objText = html.slice(braceStart, i);
const blockEnd = i;
fs.writeFileSync('/tmp/src.js', 'module.exports = ' + objText + ';');
delete require.cache[require.resolve('/tmp/src.js')];
const chordData = require('/tmp/src.js');

// ... mutate chordData.standard here ...

function serialize(value, indent) {
  const pad = '    '.repeat(indent);
  const padIn = '    '.repeat(indent + 1);
  if (Array.isArray(value)) {
    if (value.length === 0) return '[]';
    return '[' + value.map(v => serialize(v, indent + 1)).join(', ') + ']';
  } else if (typeof value === 'object' && value !== null) {
    const items = Object.keys(value).map(k => `${padIn}${JSON.stringify(k)}: ${serialize(value[k], indent + 1)}`);
    return '{\n' + items.join(',\n') + '\n' + pad + '}';
  } else {
    return JSON.stringify(value);
  }
}

let out = 'const chordData = {\n';
for (const [tuning, tuningChords] of Object.entries(chordData)) {
  out += `    ${JSON.stringify(tuning)}: {\n`;
  for (const [key, voicings] of Object.entries(tuningChords)) {
    out += `        ${JSON.stringify(key)}: [\n`;
    for (const v of voicings) out += `            ${serialize(v, 3)},\n`;
    out += `        ],\n`;
  }
  out += `    },\n`;
}
out += '};';

fs.writeFileSync(HTML_PATH, html.slice(0, start) + out + html.slice(blockEnd));
```

After running: also extract the `<script>...</script>` contents and `node --check` them to confirm no syntax errors.

## Safety notes learned the hard way
- **Never assume a "no-op" transform is safe** — a dedup pass that compares an entry against itself when a name-stripping function is a no-op will wipe out every matching entry (this happened to all `maj7` chords once). Always exclude self-matches explicitly.
- **Any auto-fix that mutes/changes frets must verify it still sounds every stated chord tone** — use a `soundsAllNotes(frets, notes)` guard and reject fixes that would silently drop a tone.
- Keep `chord-diagrams copy.html` / `chord-diagrams copy 2.html` as untouched recovery points — both have been used to rebuild from scratch after a bad bulk edit.
- After any structural change (renames, dedup, new fields), recompute derived fields if they depend on grouping — e.g. `primary` (index 13) must stay "exactly one `true` per unique chord name" whenever names change or entries are added/removed.
- Report before/after counts (total voicings, unique names, per-category breakdown) after any bulk edit so the user can sanity-check the blast radius.
