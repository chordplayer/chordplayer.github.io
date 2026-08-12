# Working Agreement

- This is a dialog-driven project: the user asks questions, Claude answers and proposes a plan.
- Do not write or edit code until the user has reviewed the plan and explicitly says to proceed.
- Once the user says to proceed on a specific, concrete request (not just general enthusiasm), act — don't re-ask for confirmation on the same request.

# Project Docs

- `DATA_SCHEMA.md` — the `chordData` voicing array format, field-by-field. Read this before touching any chord data.
- `PIPELINE.md` — the extract/mutate/serialize Node.js pattern used for all bulk data edits to `chord-diagrams.html`. Follow this pattern rather than hand-editing the embedded data.
- `DEVELOPMENT.md` — architecture and UI feature overview.

# Sole target file

`chord-diagrams.html` is the entire app (data + UI + logic, single file, no build step). `chord-diagrams copy.html` and `chord-diagrams copy 2.html` are pristine backups from earlier in the project — do not overwrite them; they've been used to recover from bad bulk-edit bugs before.
