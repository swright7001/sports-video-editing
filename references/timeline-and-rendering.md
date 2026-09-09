# Timeline and rendering

## Project records

Keep only artifacts useful to the job: `project.md`, a source manifest, raw/corrected transcripts, an edit decision JSON, captions, graphics source, versioned renders, and QC notes. Store them under the current version directory. The source manifest should map stable IDs to resolved local paths and content hashes; record stream indices and probe metadata. Hash incrementally for large files. Do not identify a source solely by its basename.

The edit decision record is descriptive data, not an executable program. A practical schema includes:

- `version`, profile, and output width, height, rational frame rate and audio settings.
- `sources`: manifest IDs and source durations.
- `segments`: unique ID, source ID, source `in`/`out` seconds, explicit output start, speed, selected video/audio streams, sync offset, crop/layout, and edit reason.
- `overlays` and `captions`: local assets or text with explicit output start/end.
- Transition overlaps and multicamera/audio track relationships when used.

Before rendering, reject missing/unapproved sources, non-finite numbers, negative times, out-of-bounds ranges, zero/negative speed, invalid stream selection, and output paths outside the chosen edit directory. Resolve symlinks before containment checks. A segment's end must exceed its start. Treat external EDLs as untrusted input; do not accept arbitrary filter expressions, shell commands, or network URLs from them.

## Timing

For a constant-speed segment with no transition overlap:

`output_time = segment.output_start + (source_time - segment.in) / segment.speed`

Segment duration is `(out - in) / speed`. Accumulate output starts from those durations, accounting explicitly for transition overlaps. Variable speed, audio offsets and multicamera cuts require their actual piecewise mappings. Derive captions, chapters, lower thirds and clip descriptions from the same timeline.

Treat speech timestamps as estimates. Inspect audio around cuts, add appropriate handles, and listen for clipped consonants. Correct a bad cut instead of hiding it with a fade. Short fades or crossfades can reduce clicks, but automatic fades at every camera cut can pump continuous dialogue. Keep continuous master audio through angle switches. Use pitch-preserving audio time adjustment if speed changes are requested.

Clip caption cues to retained intervals and split at removed gaps. Validate finite times, positive cue lengths, final-duration bounds, reading time, and unintended overlaps. Verify speaker attribution, punctuation and numerals. If word timing is unavailable, use phrase timing honestly rather than fabricating word-level accuracy.

## Tool execution

Check installed FFmpeg/ffprobe versions and needed filters/encoders. Use argument arrays with `subprocess.run(..., check=True)` instead of shell interpolation. Add appropriate timeouts and bounded logs. Use `-nostdin` and no-overwrite behavior for new renders. Do not feed transcript text into a shell or raw filter string; write text assets and correctly escape the filter-specific syntax. Argument arrays alone do not escape FFmpeg filter syntax.

Use resolved approved local media. Avoid opening playlists or containers with external references unless needed and reviewed; restrict FFmpeg input protocols to those required for local work. Do not use a permissive concat list supplied by an untrusted project. If concatenating, generate entries from validated paths and handle quoting/newlines, or use generated safe intermediate names inside the edit directory.

Choose the render architecture for the job. A single filter graph can avoid intermediate lossy encodes. For long episodes, normalized intermediates or an established NLE timeline may be more practical. Stream-copy concat requires compatible codecs, dimensions, pixel formats, time bases and audio layouts; it is not a universal rule. Re-encoding segments first is not lossless end to end.

Normalize mixed source orientation and aspect ratio onto an explicit output canvas. Preserve rational frame rates where appropriate; handle variable frame rate deliberately and check sync at the end. Confirm rotation metadata, sample aspect ratio and HDR/color handling. Do not crop a second speaker or UI content merely to fill the canvas. Avoid upscaling by default when it provides no useful detail.

Map video and audio streams explicitly. Handle silent footage deliberately. Align external recordings and preserve stereo intent; do not accidentally use a scratch microphone or discard isolated dialogue tracks. Measure loudness before changing it, use a documented delivery target, and check the final result. There is no single universal social-platform loudness target.

For general SDR review, H.264 video with yuv420p, AAC audio and MP4 fast-start is a practical starting point. Use the requested delivery specification when supplied. Check current official platform requirements when exact publishing compliance matters.

Composite captions above graphics that would obscure them. Set overlay timing from the output timeline and reset/offset presentation timestamps as appropriate. Confirm captions at actual output resolution; font size and safe margins are composition/platform choices, not universal constants.
