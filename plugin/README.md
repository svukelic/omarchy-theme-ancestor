# Animated background

Omarchy's background plugin renders the wallpaper with a QML `Image`, which
shows one frame. Qt ships `libqwebp.so`, and `AnimatedImage` — which derives
from `Image` — plays animated WebP. So an animated wallpaper needs a four-line
change to the stock plugin and no new packages, no second process, and no
video player.

[background-webp.patch](background-webp.patch) is that change: three
`Image` elements become `AnimatedImage`, and frame caching is turned off.

## Why `cache: false` is not optional

`AnimatedImage.cache` defaults to **true**, which caches decoded frames. That
is fine for a 20-frame spinner and ruinous for a 1080p wallpaper. Measured on
this machine, playing a 150-frame 1080p animation:

| `cache` | peak RSS |
| --- | --- |
| `true` (default) | 1276 MB |
| `false` | 106 MB |

The cached figure scales with frame count; the uncached one does not. A full
615-frame loop measured 116 MB with caching off. With caching on, the same loop
extrapolates to roughly 5 GB resident inside `quickshell`.

If you adapt this patch, keep that line.

## Cost

Software decode, since there is no hardware path for WebP. Measured against the
same content as h264 through mpv with `hwdec=auto`:

| | CPU |
| --- | --- |
| WebP in Qt, 1080p, cache off | 6.8% of one core |
| h264 via mpv, hardware decode | 1.5% of one core |

Both figures exclude compositing, so treat them as a ratio rather than an
absolute. The trade is roughly 4x the decode cost in exchange for not running a
second process and not disabling Omarchy's own background renderer.

## It degrades gracefully

A plain `Image` loads an animated WebP and renders its first frame — verified,
not assumed. So the wallpaper is a still on an unpatched Omarchy and animates
on a patched one. The patch is an enhancement, never a requirement, and an
Omarchy update that reverts it costs you motion rather than a black desktop.

## Install

The plugin directory is watched by the running shell, which hot-reloads on
every change. Copying into it while the shell is live can leave the shell
wedged — alive but not answering IPC, so even `omarchy-restart-shell` cannot
kill it. Stop the shell first.

```bash
omarchy plugin clone omarchy.background
```

```bash
pkill -f 'quickshell -n -p /usr/share/omarchy/shell'
```

```bash
cd ~/.config/omarchy/plugins/sirprize.background && patch -p1 < /mnt/General/Repos/omarchy-theme-ancestor/plugin/background-webp.patch
```

```bash
omarchy-restart-shell
```

Then pick the animated background from the wallpaper switcher, or:

```bash
omarchy theme bg set ~/.config/omarchy/backgrounds/ancestor/ancestor-loop.webp
```

Adjust the clone directory name if yours differs — `omarchy plugin clone` names
it after your user.

## Uninstall

```bash
pkill -f 'quickshell -n -p /usr/share/omarchy/shell'
```

```bash
rm -rf ~/.config/omarchy/plugins/sirprize.background
```

Then remove the three entries the clone added to `~/.config/omarchy/shell.json`
— `sirprize.background` from `plugins` and from `cloneSourceRestores`, and
`omarchy.background` from `disabledPlugins` — and restart the shell.

## Regenerating the loop

The source is a 12-minute music video. Its animation cycle is 1230 frames
(41.041s at 30000/1001 fps), which is not obvious: the video is cut into
alternating 5625- and 2882-frame segments whose lengths sum to 8507, so a naive
autocorrelation finds 8507 and lands on the editing pattern rather than the
animation. The real period only shows up when you search *inside* one cut-free
segment.

Cut the loop losslessly from a keyframe, dropping audio:

```bash
ffmpeg -ss 328.861867 -i source.mp4 -an -c:v copy -frames:v 1230 \
  -avoid_negative_ts make_zero -movflags +faststart loop.mp4
```

That start point is chosen so the wrap-around step is an ordinary frame step —
it sits at the 80th percentile of frame-to-frame differences in the source, and
the window avoids the video's own hard cuts.

Then halve the frame rate by exact decimation and encode. Do not use `fps=15`:
the source is 30000/1001, so resampling to a round 15 drifts and breaks the
loop. Taking every second frame keeps it exact.

```bash
ffmpeg -i loop.mp4 -vf "select='not(mod(n\,2))',setpts=N/(15000/1001)/TB" \
  -r 15000/1001 -c:v libwebp_anim -lossless 0 -q:v 75 -compression_level 6 \
  -loop 0 ancestor-loop.webp
```

GIF was measured and rejected: 181 MB for the same loop, against 11 MB for
WebP, before considering what a 256-colour palette does to dark gradients.
