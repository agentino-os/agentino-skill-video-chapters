# Changelog

All notable changes to the `video-chapters` Agentino skill are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and this skill adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-04-23

First public release for the Agentino marketplace.

### Added

- `agentino skill exec video-chapters` injects chapter atoms into an MP4 via ffmpeg's `ffmetadata` format:
  - Video + audio streams are stream-copied (`-c copy`) — zero re-encode.
  - Timebase is `1/1000` so chapter boundaries are accurate to the millisecond.
  - Chapter titles are UTF-8 with `=`, `;`, `#`, `\` properly escaped.
  - ISO 639-2 `language` tag applied to every title so YouTube, QuickTime, VLC pick the right one.
  - Auto-prepends an "Intro" chapter at `0` if the caller starts later (required by YouTube).
  - Validates ordering, non-empty titles, in-bounds start times, no duplicates.

### Tested

- 17.1 s video + 3 Italian chapters (`Introduzione 0s`, `Il contenuto 2.5s`, `Chiusura 14.6s`) → `ffprobe -show_chapters` returns all three correctly with `language=ita` tags.
- Zero findings from `agentino skill exec skill-security-check -i path=skill.yaml` (fail-on = high).

### Requires

- Agentino ≥ 1.2.0-rc.1
- `ffmpeg` (and `ffprobe`) on `PATH`
