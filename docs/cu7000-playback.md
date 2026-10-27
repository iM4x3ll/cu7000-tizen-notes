# Vemm / AVPlay notes for the CU7000

These are the playback bugs that showed up on a UN65CU7000FXZA (Tizen 9, AVPlay). They are why a “stock” Vemm build felt broken here even when the file was Direct Play.

None of this is a license to republish Vemm's source. Patch your local tree; if you publish a modified client, follow [Vemm's GPL-3](https://github.com/dragoshont/vemm).

## Defaults that survived

```
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

A local `vemm-tizen-opt-rev` key was used so older saved prefs on the TV migrated forward. Bump it when you change a default that would otherwise stay stuck in `localStorage`.

## 99% spinner, audio only

A `1080h` quality ceiling made Plex answer **transcode / HLS**. On this TV, AVPlay often never fires `onbufferingcomplete`, so the UI sat on 99% with sound and no picture.

Fix: quality **original**, Direct Play **auto**. Do not “optimize” with a 1080p cap on this panel.

Burning the Plex-selected subtitle into that HLS stream did the same stall, and left captions on after the viewer chose Off. Always send `burn=0`. Do not follow PMS `Stream.selected` for text unless the viewer picked it in *this* client.

## HUD hide

The hide timer only arms when `state === "playing"` and no overlay is open.

Two things kept the bar up:

1. Every `oncurrentplaytime` was also reported as buffering-false, which called `showHud()` and reset the 4 s timer. Gate that notify behind a real `onbufferingstart` fill flag.
2. On the **next** episode, start/buffer/state ticks called `showHud()` again. After the first hide, automatic ticks must not unhide the bar. Arm the timer once from `endStartup`, and only force-show on keys, pause, menus, and errors.

Down on the transport row still hides. Back hides the bar first, then leaves.

## Next episode froze on the last frame

`onstreamcompleted` leaves native AVPlay in **IDLE**. Calling `close()` from that state throws on this firmware. If `stopPlayer` waits on that throw (or on a “safe to leave” flag that never flips), the last frame stays forever.

`close()` must not throw. `stopPlayer` must always `leave()` — after `stop()` settles, or after a few seconds.

Prefetch `nextEpisode` during the current title so the successor metadata is already there.

## Credits cut the last minute

Plex `credits` / `final` markers on many shows start ~60 s early. `nextEpisode: countdown` started a 10 s Up Next timer at that marker and jumped.

Use **after credits**: keep a Skip Credits button, play until `STREAM_COMPLETED`, then start the next title. Skip Credits can still call `finishEpisode(true)`.

Do not auto-seek an intro marker until startup is done. A seek before the first frame crashed AVPlay on this set.

## Back hung or killed the app

`avplay.stop()` / `close()` **during `prepareAsync`** native-crashes Tizen 9. That is the “sometimes Back dies” report: the viewer leaves while the next episode is still opening.

Increment a generation to cancel the in-flight prepare, and only `stop()` + `close()` from READY / PLAYING / PAUSED / IDLE **after** prepare has finished or failed. Re-read `getState()` after `stop()`; only `close()` from IDLE.

`playback.stop()` must always return a Promise. `stopPlayer` should `try/catch` the call and still `leave()` so the detail page comes back even if native teardown is messy.

## Things that looked clever and were not

- 0.5 s buffer / 150 MB artwork: faster in a bench, worse in the living room. 1 s / 25 MB held.
- BIF preview thumbs during scrub: large decode during playback. Disabled.
- Burning “selected” subs so Off would still show “something”: caused the 99% stall and captions you cannot dismiss.
- `sdb debug` to watch logs: took down Developer Mode.

## Upstream

The right long-term home for the client fixes is a PR or issue on [dragoshont/vemm](https://github.com/dragoshont/vemm), not a silent private fork.
