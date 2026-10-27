# Vemm / AVPlay on the CU7000

Playback bugs seen on a UN65CU7000FXZA (Tizen 9, AVPlay) with a LAN Plex
server. A stock Vemm build can look broken here even when the file is Direct
Play.

This is not a license to republish Vemm. Patch a local tree; published
modifications stay [GPL-3](https://github.com/dragoshont/vemm).

Upstream report: https://github.com/dragoshont/vemm/issues/8

## Defaults that held

```text
quality          original
directPlay       auto
minBuffer        1
skipIntro        auto
skipCredits      button
nextEpisode      after_credits
continueAction   play
artworkLimitMb   25
preloadArtwork   true (home only)
enhancedGraphics false
burn subtitles   0
subtitleId       0 unless the viewer picked a track
```

A local `vemm-tizen-opt-rev` key migrated older prefs on the TV. Bump it when
a new default would otherwise stay stuck in `localStorage`.

## 99% spinner, audio only

A `1080h` quality cap made Plex answer transcode / HLS. AVPlay often never
fires `onbufferingcomplete`, so the UI sat on 99% with sound and no picture.

Use quality **original** and Direct Play **auto**. A 1080p cap is not an
optimization on this panel.

Burning the Plex-selected subtitle into that HLS stream caused the same stall
and left captions on after Off. Always send `burn=0`. Do not follow PMS
`Stream.selected` for text unless the viewer picked it in this client.

## Play bar will not hide

The hide timer arms only when `state === "playing"` and no overlay is open.

1. Every `oncurrentplaytime` was also reported as buffering-false, which called
   `showHud()` and reset the 4 s timer. Notify buffering-false once per fill.
2. On the next episode, start / buffer / state ticks called `showHud()` again.
   After the first hide, automatic ticks must not unhide the bar. Arm the timer
   once startup is done. Force-show only on keys, pause, menus, and errors.

Down on the transport row still hides. Back hides the bar first, then leaves.

## Next episode freezes on the last frame

`onstreamcompleted` leaves native AVPlay **IDLE**. `close()` from that state
throws here. If teardown waits on that throw, the last frame stays forever.

`close()` must not throw. Always leave after `stop()` or a short guard.
Prefetch `nextEpisode` during the current title.

## Credits cut the last minute

Plex `credits` / `final` markers often start about 60 s early.
`nextEpisode: countdown` started a 10 s Up Next timer at that marker.

Use **after credits**: keep Skip Credits, play until `STREAM_COMPLETED`, then
start the next title. Do not auto-seek an intro marker until the first frame
exists. That seek crashed AVPlay here.

## Back kills the app

`avplay.stop()` / `close()` during `prepareAsync` native-crashes Tizen 9.
Typical path: the next episode is still opening.

Cancel the in-flight prepare with a generation counter. Only `stop()` and
`close()` after prepare has finished or failed. Re-read `getState()` after
`stop()` and only `close()` from IDLE. `stop()` should always return a
Promise; teardown should still return to the detail page if native cleanup
fails.

## Tried and reverted

- 0.5 s buffer / 150 MB artwork
- BIF preview thumbs during scrub
- Burning “selected” subtitles
- `sdb debug` for logs
