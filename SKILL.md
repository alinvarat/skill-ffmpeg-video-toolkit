---
name: ffmpeg-video-toolkit
description: 'Edit or inspect video/audio with FFmpeg: size limits, loudness, vertical crop and audio extraction.'
---

# FFmpeg

1. Check installed `ffmpeg`/`ffprobe`: `command -v` on Debian, `Get-Command` on Windows. Report missing tools; do not install automatically.
2. Inspect input:
   `ffprobe -v error -show_entries format=duration,size,bit_rate:stream=codec_name,codec_type,width,height,r_frame_rate -of json input.mp4`
3. Preserve the master. Write a new output with `-n`. Use [recipes](references/recipes.md) only for the requested size limit, loudness, vertical crop or extraction. Replace placeholders with actual values. Copy streams only when the output container supports their codecs.
4. For a size limit, use exact bytes: MB=1,000,000; MiB=1,048,576. A positive duration and positive video bitrate are required. CRF/maxrate alone cannot guarantee bytes. Check size; retry with a lower bitrate to a new file at most twice, then report the tradeoff. Use a unique private log for two-pass encoding.
5. Probe the output; compare duration, dimensions, codecs, expected streams and bytes against the source and requested changes. Decode-check with `ffmpeg -v error -i output.mp4 -f null -`. Open sample frames for crop/orientation. For loudness, remeasure; no audio/nonfinite measurements must be reported, not filled in.
6. Report before/after duration and size plus unmet targets. State separately if no listening or visual check was performed.
