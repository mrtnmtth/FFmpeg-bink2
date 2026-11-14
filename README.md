FFmpeg-bink2
============

FFmpeg with decoders for **Bink 2** video (`.bk2`) and **Bink Audio 2**.
Upstream demuxes `.bk2` but decodes neither.

    ffmpeg -i movie.bk2 -c:v libx264 -crf 18 -c:a aac movie.mp4

Based on [`1f9b5fa5815c`](https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/1f9b5fa5815c0ea22d830da6cc4cbfcb6098def4)
(2022-05-07), because the Bink 2 decoder was never merged and does not apply to
current master. The tree is therefore years behind upstream; use a current
FFmpeg for everything else, piping between the two if needed:

    ffmpeg-bink2 -i in.bk2 -f nut -c:v rawvideo -c:a pcm_f32le - | ffmpeg -i - out.mp4

## Patches

### Bink 2 video decoder

| Change | Author | Source |
| --- | --- | --- |
| avcodec: add bink2 video decoder | Paul B Mahol \<onemda@gmail.com\> | [patchwork 35907](https://patchwork.ffmpeg.org/patch/35907/) / [series 6673](https://patchwork.ffmpeg.org/series/6673/mbox/), ffmpeg-devel 2022-05-24, unmerged |

### Upstream cherry-picks

Build fixes for current binutils, texinfo and Vulkan headers, plus Bink Audio
fixes newer than the base commit. The last is load-bearing: without it a failed
audio packet is never released and the decoder retries it forever.

| Commit | Change | Author |
| --- | --- | --- |
| [`effadce6c7`](https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/effadce6c756247ea8bae32dc13bb3e6f464f0eb) | avcodec/x86/mathops: clip constants used with shift instructions within inline assembly | Rémi Denis-Courmont |
| [`f01fdedb69`](https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/f01fdedb69e4accb1d1555106d8f682ff1f1ddc7) | doc/html: support texinfo 7.0 | Frank Plowman |
| [`eb0455d646`](https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/eb0455d64690eed0068e5cb202f72ecdf899837c) | hwcontext_vulkan: remove optional encode/decode extensions from the list | Lynne |
| [`4d9cdf82ee`](https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/4d9cdf82ee36a7da4f065821c86165fe565aeac2) | doc/t2h: Support texinfo 7.1 and 7.2 pretest | Patrice Dumas |
| [`b97e58da44`](https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/b97e58da449f78c347a88fbd09dd79a3c75e042a) | avcodec/binkaudio: Clear state on EAGAIN | Michael Niedermayer |
| [`8dcb8d1b80`](https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/8dcb8d1b808d4f8cd6444de05d8277b2e7959144) | avcodec/binkaudio: clear pts when returning more than one frame per input packet | James Almer |
| [`593e8c2c6a`](https://git.ffmpeg.org/gitweb/ffmpeg.git/commit/593e8c2c6ac0571df036c09bf103b7e58445d75d) | lavc/binkaudio: reset input packet on errors | Anton Khirnov |

### Fixes made here

| Change | Defect |
| --- | --- |
| avcodec/bink2: fix out-of-bounds reads in motion compensation | Only the block's top-left corner was bounds-checked, while the interpolation filters read up to 18 px beyond it. Segfaulted at 720x460 and 720x480. Now bounded with an edge-replicating fallback. |
| avcodec/bink2: read the column CBP flags for the 32-aligned width | Read for the 16-aligned width instead. Identical at 1920 and 3840, but at width 720 it read two flags too few and lost bitstream sync on every inter frame. |
| avcodec/binkaudio: add Bink Audio 2 support | `.bk2` audio failed as `Incomplete packet`. Backport of Peter Ross' \<pross@xvid.org\> decoder ([PR #20359](https://code.ffmpeg.org/FFmpeg/FFmpeg/pulls/20359), [patchwork 56800](https://patchwork.ffmpeg.org/patch/56800/)), with detection extended to `KB2*` container tags. |
| avcodec/bink2: filter chroma motion compensation once, rounding at the end | Filtered across, rounded, then filtered down and rounded again, drifting upwards in long runs of inter frames (magenta faces). Now one bilinear filter rounded once, matching RAD's player on 99.0 to 99.6% of pixels. |
| avcodec/bink2: fill the alpha plane of frames without alpha data | Frame flag `0x80000` means no alpha data; the plane is then filled with the frame flags' top byte. Every such frame failed as `Invalid data`. Checked against RAD Video Tools' PNG exports with alpha. |
| avcodec/bink2: widen the clamp on inter-block DC to +-2047 | Clamped at +-1023 (-128..127 as pixels), so a cut from dark to bright came out up to 62 levels short. |
| avcodec/bink2: cap block flag runs at the flags left | A damaged file could write past the row or column flag array (AddressSanitizer). |

## Building

    ./configure --disable-doc --enable-gpl
    make -j$(nproc)

Add `--disable-x86asm` if nasm is missing; the Bink 2 decoder is plain C.

## Status

136 `.bk2` files decode without errors, from 3840x2160 to 720x460, including one
with alpha and one of 98 minutes; every audio track decodes to its exact declared
length. The 438 movies of the Command & Conquer Remastered Collection (revision
`KB2j`, 80 of them with alpha) decode without errors too. Only revisions `KB2j`
and `KB2n` have been tested. Not upstreamable as-is.

## License

Unchanged from upstream: mainly LGPL with optional GPL components. The added
decoders keep their authors' headers.

---

FFmpeg README
=============

FFmpeg is a collection of libraries and tools to process multimedia content
such as audio, video, subtitles and related metadata.

## Libraries

* `libavcodec` provides implementation of a wider range of codecs.
* `libavformat` implements streaming protocols, container formats and basic I/O access.
* `libavutil` includes hashers, decompressors and miscellaneous utility functions.
* `libavfilter` provides means to alter decoded audio and video through a directed graph of connected filters.
* `libavdevice` provides an abstraction to access capture and playback devices.
* `libswresample` implements audio mixing and resampling routines.
* `libswscale` implements color conversion and scaling routines.

## Tools

* [ffmpeg](https://ffmpeg.org/ffmpeg.html) is a command line toolbox to
  manipulate, convert and stream multimedia content.
* [ffplay](https://ffmpeg.org/ffplay.html) is a minimalistic multimedia player.
* [ffprobe](https://ffmpeg.org/ffprobe.html) is a simple analysis tool to inspect
  multimedia content.
* Additional small tools such as `aviocat`, `ismindex` and `qt-faststart`.

## Documentation

The offline documentation is available in the **doc/** directory.

The online documentation is available in the main [website](https://ffmpeg.org)
and in the [wiki](https://trac.ffmpeg.org).

### Examples

Coding examples are available in the **doc/examples** directory.

## License

FFmpeg codebase is mainly LGPL-licensed with optional components licensed under
GPL. Please refer to the LICENSE file for detailed information.

## Contributing

Patches should be submitted to the ffmpeg-devel mailing list using
`git format-patch` or `git send-email`. Github pull requests should be
avoided because they are not part of our review process and will be ignored.
