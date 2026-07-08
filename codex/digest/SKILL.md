---
name: digest
description: Use when the user provides a meeting recording, transcript, chat log, Google Drive link, or asks Codex to process/digest a meeting. Uses available connectors and local tools to combine transcript, audio, video, screenshots, and chat context into a verified digest with local artifacts. Project-agnostic and privacy-first.
---

# Digest Skill For Codex

Process a meeting, lesson, interview, or recorded working session into a verified
digest. Treat the recording, audio, transcript, chat, and screenshots as
complementary sources.

Run autonomously. Pause only when source access fails, a required tool is
missing, the output directory is ambiguous, or a destructive/irreversible choice
would be required.

## Operating Defaults

- Match the user's language in all generated artifacts.
- Prefer connector access for Google Drive when available; fall back to browser,
  local files, or user-provided downloads.
- If the user asks to download the recording, download it before analysis.
- Prefer existing transcripts over re-transcribing audio.
- Use video and screenshots for context, especially slides, product demos,
  charts, visible errors, timestamps, and UI states.
- Do not generate PDFs by default. Generate PDFs only when explicitly requested.
- Never assume a specific person, email, Drive account, local path, project, or
  organization.

## Parameters

Infer or ask once:

- `PROJECT`: short generic project/session label.
- `OUTPUT_DIR`: default to `meeting-digests/<PROJECT>/<DATE>` under the current
  working directory unless the user or current project has a clear convention.
- `STAKEHOLDER`: optional audience for the executive summary.
- Source set: recording, transcript, captions, chat, screenshots, or links.

If `OUTPUT_DIR` is inside a git repository, ensure generated artifacts are
ignored before writing private meeting content.

## Source Acquisition

Accept any combination of:

- Google Drive file or document links.
- Local `.mp4`, `.mov`, `.webm`, `.mp3`, `.m4a`, `.wav`, `.srt`, `.vtt`, `.txt`,
  `.md`, `.docx`, or Google Docs export.
- Chat logs and screenshot folders.

For Drive links:

1. Extract the file ID when useful.
2. Inspect metadata before downloading large media.
3. Look for sibling artifacts when access allows: recording, transcript,
   captions, chat log, and related docs.
4. Do not put private Drive URLs into reusable docs, examples, commits, or issue
   comments.

When the user supplies multiple links, classify each source before processing.

## Media Inspection

When video or audio exists:

- Use `ffprobe` to collect duration, stream list, codec, resolution, audio, and
  caption/subtitle availability.
- Extract embedded captions first if present.
- Extract audio only if transcription is needed:
  `ffmpeg -i <recording> -vn -ar 16000 -ac 1 audio.wav -y`
- Use a transcription tool or API available in the current environment if no
  transcript/captions exist.
- Preserve uncertainty from ASR. Do not silently "fix" names, numbers, or terms.

## Visual Evidence

When video or screenshots are available:

- Create `screenshots/`.
- Capture timestamped screenshots for the opening context, topic changes,
  slides, charts, product demos, visible errors, decisions, and action-item
  moments.
- Use stable names such as `screenshots/00-12-34-dashboard.png`.
- Inspect screenshots directly when the environment supports image viewing.
- Create `visual-references.md` with:
  - timestamp
  - screenshot path
  - visible context
  - digest claims supported by that visual

Exclude or redact screenshots that expose secrets, credentials, private URLs,
personal data, customer data, or unrelated private context.

## Artifacts

Write artifacts under `OUTPUT_DIR`:

1. `source-metadata.json`: source labels, source types, durations, and extraction
   notes without secrets.
2. `transcript-raw.txt`: raw transcript, captions, or ASR output.
3. `transcript-clean.txt`: cleaned transcript preserving meaning.
4. `screenshots/`: timestamped screenshots when visual context exists.
5. `visual-references.md`: screenshot index and visual evidence.
6. `digest.md`: full verified digest.
7. `executive-summary.md`: concise stakeholder summary.
8. `technical-analysis.md`: evidence-backed analysis.
9. `handoff.md`: next-session plan.

Do not commit these artifacts unless the user explicitly confirms they are
sanitized examples.

`source-metadata.json` allowed fields:

- `source_label`, `source_type`, `mime_type`
- `duration_seconds`, `stream_summary`, `screenshot_count`
- `transcript_method`, `caption_method`, `notes`

Do not store full Drive URLs, full file IDs, personal emails, access tokens,
absolute local paths, private account identifiers, or raw credentials. Store
original filenames only when the user confirms they are safe; otherwise use
generic labels such as `recording-1` or `transcript-1`.

## Digest Template

```markdown
# Digest - <PROJECT> - <DATE>

## Source Metadata
- Recording:
- Transcript:
- Audio:
- Chat:
- Visual references:

## Executive Summary

## Decisions
- Mark each item as `closed decision`, `intention`, `hypothesis`, or
  `to validate`.

## Action Items
- Owner:
- Task:
- Priority:
- Due date:
- Source anchor:

## Important Discussion

## Technical / Domain Notes

## Visual References

## Risks, Gaps, And Unconfirmed Items

## Follow-Up Plan
```

Use source anchors for important claims: transcript timestamp, short quote,
visual timestamp, or source metadata. Keep quotes short and only when needed for
verification.

## Verification

Do not skip verification.

Before finalizing:

- Re-check numbers, names, dates, deadlines, URLs, roles, and action owners.
- Compare `digest.md`, `executive-summary.md`, and `technical-analysis.md`
  against `transcript-raw.txt`, `transcript-clean.txt`, and
  `visual-references.md`.
- Verify that visual claims are backed by the screenshot at the cited timestamp.
- Distinguish decision vs intention vs open question.
- Mark unsupported content as `unconfirmed`.

If subagents are available, use isolated reviewers:

- Transcript reviewer: facts, numbers, names, decisions, omissions.
- Visual reviewer: screenshots, timestamps, slide/demo/UI interpretation.

Fix the artifacts after review before reporting completion.

## Final Report

Tell the user briefly:

- Output directory.
- Files created.
- Which sources were used: recording, audio, transcript, chat, screenshots.
- Main action items and recommended next step.
- Any low-confidence or unconfirmed facts.

## Privacy And Portability

- Do not hardcode user identity, account email, Drive remote names, local absolute
  paths, project names, or private URLs.
- Do not publish generated meeting artifacts.
- Keep examples generic and sanitized.
- Preserve public authorship as ZaMpA.
