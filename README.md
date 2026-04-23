# video-chapters

> **Add YouTube / QuickTime chapter markers to an existing MP4 — no re-encode, just metadata. Chapters show up as a timeline scrubber in every modern player and video platform.**

An Agentino skill that injects chapter atoms into an MP4 container using ffmpeg's `ffmetadata` format. The video and audio streams are stream-copied (zero quality loss), only the metadata is rewritten. Chapter titles support any UTF-8 string and carry an ISO 639-2 language tag so YouTube picks the right one.

## Install

Requires [Agentino](https://github.com/dagoSte/agentino) ≥ `1.2.0-rc.1` and `ffmpeg`.

```bash
brew install ffmpeg                                                    # macOS
# sudo apt install ffmpeg                                              # Debian / Ubuntu

agentino marketplace install agentino-os/agentino-skill-video-chapters
agentino skill show video-chapters
```

## Use

```bash
agentino skill exec video-chapters --input-json '{
  "video_path": "/tmp/talk.mp4",
  "chapters": [
    {"title": "Introduction", "start_s": 0},
    {"title": "The core idea", "start_s": 120},
    {"title": "Demo time", "start_s": 420},
    {"title": "Q&A", "start_s": 900}
  ],
  "language": "eng",
  "output_path": "/tmp/talk.chaptered.mp4"
}'
```

### Example output

```json
{
  "output_path": "/tmp/talk.chaptered.mp4",
  "duration_s": 1034.2,
  "chapter_count": 4,
  "chapters": [
    {"title": "Introduction", "start_s": 0.0,   "end_s": 120.0},
    {"title": "The core idea", "start_s": 120.0, "end_s": 420.0},
    {"title": "Demo time",    "start_s": 420.0, "end_s": 900.0},
    {"title": "Q&A",          "start_s": 900.0, "end_s": 1034.2}
  ],
  "language": "eng"
}
```

Verify with `ffprobe -show_chapters /tmp/talk.chaptered.mp4`.

## Use with agentino run

```bash
agentino run "add chapters to /tmp/talk.mp4: Intro at 0s, Main at 120s, Demo at 420s, Q&A at 900s. Save to /tmp/talk.chaptered.mp4"
```

The LLM planner picks this skill because the prose mentions "add chapters" + timestamped labels.

## Inputs

| Input | Type | Default | Description |
|---|---|---|---|
| `video_path` | string | **required** | Source video. |
| `chapters` | array | **required** | Ordered list of `{title: str, start_s: float}`. End time is derived from the next chapter's start (or the video duration for the last chapter). If the first chapter starts after 0 s, an implicit "Intro" chapter is prepended. |
| `output_path` | string | `<video>.chaptered.<ext>` | Destination MP4. |
| `language` | string | `eng` | ISO 639-2 language tag stored with each title (`ita`, `fra`, `deu`, …). |

## Outputs

- **`output_path`** — absolute path to the chaptered MP4.
- **`duration_s`** — total video duration from `ffprobe`.
- **`chapter_count`** — number of chapters written (may be higher than the input if an implicit Intro was prepended).
- **`chapters`** — resolved chapter list with derived `end_s` values.
- **`language`** — the language tag applied to every chapter title.

## How it works

1. **Validate the chapters list** — titles non-empty, `start_s` sorted strictly ascending, all within `[0, duration)`; duplicates refused.
2. **Auto-prepend** an "Intro" chapter at `0` if the caller didn't start at zero (YouTube requires the first chapter to be at 00:00).
3. **Derive end times** — each chapter runs until the next one starts; the last chapter runs to the end of the video.
4. **Write an `ffmetadata` file** in a tempdir — `TIMEBASE=1/1000`, `START`/`END` in milliseconds, escaped titles (`=`, `;`, `#`, `\` all handled).
5. **Remux via `ffmpeg`** — `-map_metadata 1 -map_chapters 1 -c copy -movflags +faststart`. No re-encode on video or audio.

## Safety

- `sandbox_level: none` (ffmpeg discovery).
- `network_access: false`.
- `file_access: read-write`.
- `agentino skill exec skill-security-check -i path=skill.yaml` → zero findings.

## License

MIT — see [`LICENSE`](./LICENSE).
