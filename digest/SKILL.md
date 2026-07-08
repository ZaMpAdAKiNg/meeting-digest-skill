---
name: digest
description: 'Use when the user says a meeting recording is in their Google Drive, pastes a Google Drive link to a recording/transcript, or asks to "process/digest a meeting". Downloads the recording, transcribes it, and produces a full meeting digest (executive summary, decisions, action items, executive PDF) + saves canonical artifacts. Project-agnostic. Triggers: "the meeting is in my drive", "digest the meeting", "process this recording", "digest", or a pasted drive.google.com/file/d/... link.'
---

# Meeting Digest Pipeline

You are processing a meeting recording end-to-end. Run the whole pipeline
**autonomously** — don't ask the user to confirm each step. Only pause if a
prerequisite is missing, the file can't be located/identified, or there's an
irreversible decision.

## Parameters (detect or ask once, then remember for the project)
- **PROJECT** — short project name (infer from cwd/repo; ask if unclear).
- **BASE_DIR** — where digests live. Default `~/Documents/<PROJECT>/Meetings`. Ask if the project already has a convention.
- **STAKEHOLDER** — who the executive summary is for (CEO / client / boss). Ask once.
- Document language = match the user.
- If the project has a meeting-pipeline doc (e.g. `~/Documents/<PROJECT>/_pipelines/process-meeting.md` or a project handoff), READ it first — it overrides these defaults.

## Prerequisites (check; if missing, tell the user how to install)
`rclone` (with a Google Drive remote, scope=drive — find via `rclone listremotes`; if >1, ask which → call it `<REMOTE>`), `ffmpeg`+`ffprobe`, `pandoc`, Google Chrome (PDF via headless), `python3`.

## Step 1 — Locate the recording
- **Link given**: extract FILE_ID from `https://drive.google.com/file/d/<FILE_ID>/view`.
- **"It's in my Drive"**: list the recordings folder (Google Meet → `Meet Recordings/`; Zoom/others vary) sorted by ModTime desc, pick the most recent, confirm in one line:
  `rclone lsjson "<REMOTE>:Meet Recordings"` → sort by ModTime.

A Meet call can produce up to **3 independent artifacts** (recording video and transcription are separate toggles — one may exist without the other):
- (a) **video** `.mp4` — `<code> (date)` — hundreds of MB
- (b) **spoken transcript** — `<code> (date) - Transcript.docx` — diarized text
- (c) **chat log** — `<code> (date) - Chat` — a few KB
**Inspect name/size/mime BEFORE downloading** — never pull a 700MB video unnecessarily.

## Step 2 — Get the text (preference order)
1. **Spoken transcript `.docx`** (best — clean, diarized):
   `rclone backend copyid <REMOTE>: <FILE_ID> /tmp/<PROJECT>-meet-<DATE>/transcript.docx`
   `pandoc transcript.docx -t plain -o transcript.txt`
2. **mp4 with embedded captions**: `ffprobe ... ` to detect a `subtitle` stream → `ffmpeg -i meeting.mp4 -map 0:s:0 captions.srt -y` → clean.
3. **Audio-only fallback** (heavier): `ffmpeg -i meeting.mp4 -vn -ar 16000 -ac 1 audio.mp3 -y` → transcribe via Whisper or an audio-capable LLM API.

**Pitfalls (always handle):**
- Filenames may contain special chars (narrow no-break space) → **download by FILE ID** (`rclone backend copyid`), not by path.
- A native Google Doc shows `Size: -1` → needs export (`--drive-export-formats txt`) or copyid+pandoc. If mime is already real `.docx`, copyid grabs the binary.
- Meet transcripts **auto-delete from Drive after ~90 days** → always save a local copy.
- If the **chat log** exists, download it too — it has the EXACT URLs/names pasted in the call, useful to fix ASR errors (the transcript mis-hears names/domains).
- **Clean up**: merge consecutive same-speaker lines into paragraphs (small python) → `transcript-clean.txt`.

## Step 3 — Read the full transcript, then generate artifacts
Create `BASE_DIR/<DATE>/` and write:
1. `transcript-clean.txt` + `transcript-raw.txt` (+ keep the original `.docx`/`.srt`)
2. `digest.md` — full internal doc (template below)
3. `executive-summary.md` — short version for STAKEHOLDER (no excess technical detail)
4. `technical-analysis.md` — with verbatim QUOTES anchored to the transcript
5. `handoff.md` — numbered action plan for the next work session

**digest.md template:**
```
# Digest — Meeting <PROJECT> <DATE>
Metadata: date/time · participants · duration · source (video? transcript?)
## Executive Summary (3-4 paragraphs)
## Decisions Made (numbered; mark CLOSED DECISION vs intention/to-validate)
## Action Items (by owner; with priority)
## Insights / Notes (business · technical · process)
## Comparison with the previous meeting (resolved / pending / new) — if any
## NEW Items (for future conversations)
```

## Step 4 — Adversarial verification (don't skip)
Cross-check digest/analysis against the RAW transcript:
- **Numbers** (values, %, deadlines) — any swapped/invented?
- **Names & roles** — any misattribution of who said/is what?
- **Decision vs intention** — don't label "decided" what was "to validate".
- **Omissions** — any decision/action-item left out?
If subagents are available, delegate this to an isolated adversarial reviewer. Fix everything BEFORE generating PDFs. If a fact isn't clearly in the source, mark it "unconfirmed" rather than asserting it.

## Step 5 — Generate PDFs (Chrome headless — faithful accents/emoji)
Use a branded CSS. **If the CSS has the date in the footer (`@page @bottom-left`), UPDATE it per meeting** (common bug: inheriting the previous meeting's date). Avoid `background-clip:text` with gradient (leaks in Chrome).
```
pandoc executive-summary.md -o tmp.html --css=_style.css --standalone --embed-resources
# Then print to PDF with headless Chrome. The Chrome/Chromium binary path is OS-specific:
#   macOS:   "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
#   Linux:   google-chrome   (or chromium-browser / chromium)
#   Windows: "C:\Program Files\Google\Chrome\Application\chrome.exe"
<chrome> --headless --disable-gpu \
  --no-pdf-header-footer --print-to-pdf="Meeting <PROJECT> <DATE> — for <STAKEHOLDER>.pdf" "file://$PWD/tmp.html"
```
Detect the binary for the current OS (fall back to `chromium`/`chromium-browser` on Linux). Generate 2 PDFs: executive (for STAKEHOLDER) + complete (internal). Validate with `pdftotext` that no character is broken.

## Step 6 — Memory
Save one short memory per meeting (key decisions + action items + what changes) and update the project's memory index. Don't duplicate what's already in the docs.

## Step 7 — Report back
Tell the user, briefly: where the files are, WHICH PDF to send to the STAKEHOLDER, the main action items + recommended next step, and anything needing their confirmation (e.g., a number the transcript left unclear).

## Behavior
- Autonomous; no step-by-step confirmation.
- Ask only if: missing prerequisite, file not found/identified, or irreversible decision.
- Honesty: if only a transcript exists (no video) or vice-versa, say so and proceed with what's available. Never invent content not in the source.
- Reuse the CSS/structure from a previous meeting of the same project (just change the date).
