---
name: digest
description: Use when the user provides a meeting recording, transcript, chat log, Google Drive link, or asks to process/digest a meeting. Produces a verified meeting digest with transcript artifacts, visual references when video is available, executive summary, technical analysis, handoff, and optional PDFs. Project-agnostic and privacy-first.
---

# Meeting Digest Pipeline For Claude Code

Process a meeting recording end-to-end. Run autonomously and pause only when a
source cannot be accessed, a prerequisite is missing, the output location is
ambiguous, or a destructive/irreversible choice would be required.

## Parameters

Detect these from context or ask once:

- `PROJECT`: short project name. Use a generic label if the meeting is not tied
  to a repo or project.
- `OUTPUT_DIR`: default to `meeting-digests/<PROJECT>/<DATE>` under the current
  working directory unless the user or project has a clear convention.
- `STAKEHOLDER`: audience for the executive summary, if any.
- Output language: match the user's language.
- Existing process docs: if the active project has a meeting-processing guide,
  read it first and treat it as an override.

Never hardcode a person, company, local path, Drive account, remote name, or
private project into this skill.

## Prerequisites

Check what is actually needed for the chosen source path:

- `ffmpeg` and `ffprobe` for video/audio inspection, audio extraction, captions,
  and screenshots.
- A Drive access path when the source is in Google Drive: Claude connector,
  authenticated browser, or `rclone`.
- `pandoc`, `textutil`, or another document converter for `.docx` transcripts.
- A transcription path when no transcript or captions exist.
- Chrome and `pdftotext` only when PDF generation is requested.
- `python3` for small cleanup scripts when useful.

If a prerequisite is missing, tell the user the smallest install step for their
platform and stop before partial processing.

## Step 1: Locate And Inspect Sources

Accept any combination of:

- Google Drive recording link.
- Local video or audio file.
- Transcript document or text file.
- Captions file.
- Chat log.
- Existing screenshot folder.

For Drive links, extract the file ID and inspect metadata before downloading
large media. A meeting may have separate recording, transcript, captions, and
chat artifacts; look for all of them when access allows.

Generic Drive recipe:

1. Inspect metadata with the available connector, browser session, or `rclone`
   before downloading.
2. If using `rclone`, list candidate folders with `rclone lsjson` or `rclone lsf`
   and ask once if multiple remotes/folders could match.
3. Prefer file-ID based download/export when the tool supports it, using generic
   placeholders such as `<REMOTE>`, `<FILE_ID>`, and `<OUTPUT_DIR>`.
4. For native Google Docs transcripts, export to text or `.docx` before cleanup.
5. Look for sibling transcript, captions, and chat artifacts near the recording.

Do not write private Drive URLs into public docs, examples, commits, or issue
comments. Generated private artifacts may include source references only for the
user's local use.

## Step 2: Build Text Sources

Preference order:

1. Existing diarized transcript document or text file.
2. Embedded captions or sidecar captions.
3. Audio transcription from the recording.

Acceptable transcription paths:

- Local Whisper or another installed local ASR tool.
- A harness-provided audio transcription capability.
- A user-approved API transcription path.

If no transcription path is available, stop and report the missing capability
instead of producing a transcript-free digest.

Keep both:

- `transcript-raw.txt`: source text as extracted.
- `transcript-clean.txt`: cleaned text with obvious formatting noise removed and
  consecutive same-speaker fragments merged when safe.

If speaker labels, names, numbers, or timestamps are uncertain, preserve that
uncertainty instead of silently correcting it.

## Step 3: Build Visual Context

When video or screenshots are available, use them as evidence:

- Run `ffprobe` to capture duration, streams, resolution, and audio/caption
  presence.
- Extract screenshots at the opening frame, major topic changes, slide/demo
  transitions, visible errors, dashboards, decisions, and action-item moments.
- Name screenshots with stable timestamps, for example
  `screenshots/00-12-34-topic.png`.
- Create `visual-references.md` with timestamp, screenshot file, and what the
  visual source confirms.

Exclude or redact screenshots that expose secrets, personal data, customer data,
private URLs, credentials, tokens, or unrelated private context.

## Step 4: Generate Artifacts

Write artifacts under `OUTPUT_DIR`:

1. `source-metadata.json`
2. `transcript-raw.txt`
3. `transcript-clean.txt`
4. `screenshots/` when video context exists
5. `visual-references.md` when visual context exists
6. `digest.md`
7. `executive-summary.md`
8. `technical-analysis.md`
9. `handoff.md`

`source-metadata.json` allowed fields:

- `source_label`, `source_type`, `mime_type`
- `duration_seconds`, `stream_summary`, `screenshot_count`
- `transcript_method`, `caption_method`, `notes`

Do not store full Drive URLs, full file IDs, personal emails, access tokens,
absolute local paths, private account identifiers, or raw credentials. Store
original filenames only when the user confirms they are safe; otherwise use
generic labels such as `recording-1` or `transcript-1`.

`digest.md` structure:

```markdown
# Digest - <PROJECT> - <DATE>

## Source Metadata
## Executive Summary
## Decisions
## Action Items
## Important Discussion
## Visual References
## Risks And Open Questions
## Follow-Up Plan
```

Anchor important claims to transcript timestamps, short quotes, screenshot
timestamps, or source metadata. Keep quotes short and only when they materially
improve verification.

## Step 5: Adversarial Verification

Before finalizing:

- Cross-check numbers, names, roles, dates, deadlines, and URLs.
- Distinguish closed decisions from intentions, hypotheses, and next steps.
- Check that action items have owner, task, priority, and due date when stated.
- Compare visual references against the digest so screenshots support the claims
  they are cited for.
- Mark unsupported claims as `unconfirmed` instead of asserting them.

If subagents or reviewers are available, ask one reviewer to verify transcript
facts and another to verify visual/timestamp evidence. Fix the artifacts before
reporting completion.

## Step 6: Optional PDFs

Generate PDFs only if requested or if the user's workflow requires them:

- Use the markdown artifacts as source.
- Reuse a user-provided CSS when available; otherwise use a simple readable
  style.
- Validate generated PDFs with text extraction when possible.
- Never let PDF generation block delivery of the markdown digest unless the PDF
  is the requested deliverable.

## Step 7: Report Back

Briefly report:

- Output directory.
- Main digest file.
- Whether video, audio, transcript, chat, and screenshots were used.
- Main action items.
- Any unconfirmed or low-confidence facts.
- Recommended next step.

## Privacy Rules

- Do not commit or publish generated meeting artifacts.
- Do not include personal emails, private Drive links, local absolute paths,
  private project names, credentials, or customer data in reusable docs.
- Keep examples generic.
- Keep public authorship as ZaMpA.
