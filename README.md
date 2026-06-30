# Meeting Digest — a Claude Code Skill

Turn a meeting recording into a clean, verified digest — automatically.

This is an [Agent Skill](https://docs.claude.com/en/docs/claude-code/skills) for
[Claude Code](https://www.claude.com/product/claude-code). Point it at a Google
Drive recording (or paste a Drive link) and it runs the whole pipeline end-to-end:
locate the file → pull the transcript → write a full digest, an executive summary,
a technical analysis and a handoff → **adversarially fact-check it against the raw
transcript** → render branded PDFs.

It's **project-agnostic** — no hardcoded names, paths, or org details. You set a
few parameters the first time and it remembers them per project.

## What it produces

For each meeting, under `BASE_DIR/<DATE>/`:

| File | Purpose |
|------|---------|
| `transcript-clean.txt` / `transcript-raw.txt` | cleaned + original transcript |
| `digest.md` | full internal doc: summary, decisions, action items, insights |
| `executive-summary.md` | short version for a stakeholder (CEO / client / boss) |
| `technical-analysis.md` | analysis with verbatim quotes anchored to the transcript |
| `handoff.md` | numbered action plan for the next session |
| 2× PDF | executive (for the stakeholder) + complete (internal) |

## Why it's different from "just summarize the transcript"

- **Prefers the diarized `.docx` transcript** over re-transcribing audio (cleaner, knows who said what), with an `ffmpeg` captions path and a Whisper audio fallback.
- **Adversarial verification step** — a second pass cross-checks numbers, names, and decision-vs-intention against the raw transcript, and marks anything not clearly in the source as *unconfirmed* instead of inventing it.
- **Handles real-world Drive gotchas** — special characters in filenames, native Google Docs reporting `Size: -1`, and Meet transcripts that auto-delete after ~90 days.
- **Faithful PDFs** via headless Chrome (accents and emoji survive), validated with `pdftotext`.

## Prerequisites

The skill checks for these and tells the user how to install any that are missing:

- [`rclone`](https://rclone.org/) with a Google Drive remote (`scope=drive`)
- `ffmpeg` + `ffprobe`
- [`pandoc`](https://pandoc.org/)
- Google Chrome (used headless for PDF rendering)
- `python3`

On macOS: `brew install rclone ffmpeg pandoc python`

## Install

Skills live in `~/.claude/skills/<name>/SKILL.md`. Copy the `digest/` folder there:

```bash
git clone https://github.com/<your-user>/meeting-digest-skill.git
cp -r meeting-digest-skill/digest ~/.claude/skills/digest
```

Restart Claude Code (or start a new session) so the skill is discovered.

> You can also drop it into a project at `.claude/skills/digest/` to scope it to one repo, or ship it inside a plugin.

## Use

Just talk to Claude Code naturally — the skill triggers on intent:

- "The meeting recording is in my Drive, make the digest."
- Paste a `https://drive.google.com/file/d/.../view` link.
- "Process this recording" / "digest".

It runs autonomously and only pauses if a prerequisite is missing, it can't
identify the file, or it hits an irreversible decision.

## How it works

See [`digest/SKILL.md`](digest/SKILL.md) for the full pipeline. In short:

1. **Locate** the recording in Drive (inspect name/size/mime *before* downloading a 700MB video).
2. **Get the text** — diarized `.docx` → embedded captions → audio + Whisper, in that order.
3. **Generate** the digest, executive summary, technical analysis, and handoff.
4. **Verify** adversarially against the raw transcript.
5. **Render** two PDFs and validate them.
6. **Remember** the key decisions and report back which PDF to send.

## Notes & assumptions

- Defaults to Google Meet's `Meet Recordings/` folder layout; Zoom and others vary — adjust Step 1 for your provider.
- `BASE_DIR` defaults to `~/Documents/<PROJECT>/Meetings`; change it to your convention.
- The PDF step references a branded `_style.css` — bring your own; an example is in [`examples/_style.css`](examples/_style.css).

## License

MIT — see [LICENSE](LICENSE).
