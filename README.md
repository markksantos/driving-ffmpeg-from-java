# Driving FFmpeg from Java

Example code for the foojay.io article ["Driving FFmpeg from Java Without Deadlocks or Broken Files"](https://foojay.io/today/driving-ffmpeg-from-java/): running `ffmpeg` and `ffprobe` from `ProcessBuilder` on JDK 21+ without the stderr pipe deadlock, parsing ffprobe JSON into records, reading live progress, cancelling cleanly and running encodes in parallel.

Each example is a single source file. Most run with `java Name.java`; the two that parse JSON (`Probe.java`, `Loudnorm.java`) declare a Jackson dependency in a `//DEPS` comment and run with [JBang](https://www.jbang.dev/).

## Requirements

- JDK 21 or later (`CloseTest.java` needs JDK 26 or later)
- `ffmpeg` and `ffprobe` on the `PATH`
- JBang for `Probe.java` and `Loudnorm.java`

## Test media

`./make-media.sh` writes every input the examples use into `media/` with FFmpeg's own generators, so there is nothing to download. It also creates an empty `out/` for the encodes.

## Examples

| File | What it shows | Run |
| --- | --- | --- |
| `Deadlock.java` | A full stderr pipe stalls `waitFor()` on long inputs | `java Deadlock.java media/long.mp4` |
| `Drained.java` | Reading stdout and stderr on separate threads fixes it | `java Drained.java media/long.mp4` |
| `Probe.java` | ffprobe JSON into records | `jbang Probe.java media/short.mp4 media/ntsc.mov` |
| `Progress.java` | Live progress from `-progress pipe:1` | `java Progress.java media/minute.mp4 out/minute-x264.mp4` |
| `Cancel.java` | Stopping an encode: `q`, `destroy()`, `destroyForcibly()` | `java Cancel.java media/minute.mp4` |
| `CloseTest.java` | `Process.close()` on JDK 26 waits; it does not cancel | `java CloseTest.java media/minute.mp4 out/close.mp4` |
| `Batch.java` | Parallel encodes with a concurrency limit | `java Batch.java 1,2,4,8 media/clip{1..8}.mp4` |
| `Carriers.java` | What blocked process reads cost virtual threads | `java Carriers.java 100 wait` and `java Carriers.java 100 read` |
| `Loudnorm.java` | Two-pass loudness normalization to -14 LUFS | `jbang Loudnorm.java media/tone.mp4 out/tone-14.mp4` |

## License

MIT, see [LICENSE](LICENSE).
