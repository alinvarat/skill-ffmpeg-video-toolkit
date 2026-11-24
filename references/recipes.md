# FFmpeg recipes

Read only the operation requested. Substitute computed numeric values for placeholders before running.

## Hard size limit
Use the user's limit in **bytes** (10 MB = 10,000,000 bytes; 10 MiB = 10,485,760). For duration D seconds, allow container headroom:
```
total_kbps = target_bytes * 8 / D / 1000 * 0.95
video_kbps = total_kbps - audio_kbps
```
If D is missing/zero or video_kbps is nonpositive, resolve that constraint instead of running an invalid command. CRF and maxrate alone do not guarantee file size; use average bitrate or two-pass encoding when needed.
```bash
ffmpeg -n -i input.mp4 -c:v libx264 -preset slow -b:v VIDEO_KBPSk -c:a aac -b:a 128k -movflags +faststart output.mp4
```
Check the actual byte count. If oversized, retry to a new temporary output with a reduced bitrate, at most twice; if it still misses, report the quality/size tradeoff rather than loop indefinitely. For two-pass mode, use a private unique pass-log path.

## Loudness (-14 LUFS unless requested otherwise)
First measure, then substitute all measured values in the second pass:
```bash
ffmpeg -i input.mp4 -af loudnorm=I=-14:LRA=7:TP=-1.5:print_format=json -f null -
ffmpeg -n -i input.mp4 -af "loudnorm=I=-14:LRA=7:TP=-1.5:measured_I=VALUE:measured_LRA=VALUE:measured_TP=VALUE:measured_thresh=VALUE:offset=VALUE:linear=true" -c:v copy -c:a aac -b:a 192k output_norm.mp4
```
If there is no audio or measurements are nonfinite (such as silent input), report it rather than inventing loudness values. Re-measure the output to check the requested target.

## Vertical 9:16
Choose crop only when it preserves the subject. This scale-then-crop works with both portrait and landscape input:
`-vf "scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920,setsar=1"`
For background fill:
`-vf "split[a][b];[a]scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920,boxblur=20:5[bg];[b]scale=1080:1920:force_original_aspect_ratio=decrease[fg];[bg][fg]overlay=(W-w)/2:(H-h)/2,setsar=1"`
Re-encode filtered video, for example `-c:v libx264 -crf 18 -preset slow`, and preserve compatible audio. Inspect sample frames for cropping and orientation.

## Audio
- Extract MP3: `ffmpeg -n -i input.mp4 -vn -c:a libmp3lame -q:a 2 audio.mp3`.
- Copy audio only to a compatible container (`.aac`/`.m4a` for AAC).
- Detect silence: `ffmpeg -i input.mp4 -af silencedetect=noise=-30dB:d=0.4 -f null -`.

