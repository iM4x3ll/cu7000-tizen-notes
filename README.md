# Samsung CU7000 / Tizen 9 living-room notes

A working living-room setup on a **Samsung UN65CU7000FXZA** (Tizen 9, armv7): a quiet custom home screen, a local Plex client that actually Direct Plays, and the TV features that are worth keeping.

This is a field notes repo, not a product. It records what worked on one Crystal UHD, 2023-class set after a lot of wrong turns. It is not a root kit, not a Home-button exploit, and not a dump of someone else's client.

**Tested on one TV.** Treat every other model, year, and firmware as unproven.

## What we kept

- AirPlay 2
- iPhone Remote
- Samsung TV Plus
- YouTube
- Official Plex (left installed; the living-room player is Vemm)

## What we used

| Piece | Upstream | Role on this TV |
| --- | --- | --- |
| Arn0 Launcher (`Arn0Lnchr0.Launcher`) | Quiet Tizen home screen (local `arn0-tv` tree; related idea: [CleanFrame](https://github.com/nthnclrk/CleanFrame)) | Apple TV–style dock: Vemm, TV Plus, YouTube, AirPlay, Netflix, Disney+, Prime, Spotify, Stremio, Live TV, HDMI 1–3 |
| [Vemm Preview](https://github.com/dragoshont/vemm) (`VemmPrevw0.VemmPreview`) | GPL-3 Plex-compatible Tizen client | AVPlay + Plex Direct Play |
| Official Developer Mode | Samsung | Sideload signed `.wgt` only |

Home watcher (`Arn0Lnchr0.Watcher`) plus **Autorun Last App** is the Route B remap. **Autorun Last App ignores sideloaded packages on this firmware**, so the watcher still matters. Samsung's own Home stays installed.

## What we refused

- SSH / port 22
- Root or archive-root
- Home remaps that use SVE-2025-50109 or similar
- `sdb shell 0 debug` (it hung the Developer Mode session on this set)

If a guide tells you to root a 2023 Samsung to replace Home, it is the wrong guide for this TV.

## Install path that actually works

Retail `sdb` on Tizen 9 only does a handful of things reliably: `applist`, `getduid`, `install` / `uninstall`. A plain `sdb install` often **only pushes** the file.

What worked here:

1. Put the TV in Developer Mode and bind the PC.
2. Sign the unsigned app with the **same author certificate** you will keep using (`Apps2Samsung` / Tizen author cert for this DUID).
3. `sdb push` the signed `.wgt` to `/home/owner/share/tmp/sdk_tools/`.
4. Install with `sdb shell 0 vd_appinstall <appid> <remote-wgt-path>`.
5. Launch with `sdb shell 0 was_execute <appid>` or from the app list.

Do not rotate the author certificate on an update. A mismatch can force an uninstall and wipe the app's local settings.

Store apps (official Plex, TV Plus, YouTube) did **not** uninstall cleanly through `sdb` on this firmware. Leave them, or remove them from the TV's own menus.

## Boot / Home reality

- **Autorun Last App** does not treat a sideloaded launcher as a last app.
- If the TV still wakes into Smart Hub, turn **Start with Smart Hub Home** off and **Autorun Samsung TV Plus** off on the TV itself.
- The software Home watcher is the thing that brings Arn0 back after you press Home. It is not instant; a 1.5 s poll was enough here.
- Keep a restore script that only uninstalls **your** packages (`VemmPrevw0`, `DclHome001`, …). Do not wipe Samsung Home, AirPlay, or TV Plus.

## Vemm on this CU7000

Vemm is still the best AVPlay Plex client we compared for this set (official Plex, Flixor, and the usual sideload names included). The stock defaults are not CU7000-safe.

### Settings that stayed stable

- Quality **original** (not a 1080p cap)
- Direct Play **auto**
- Min buffer **1 s**
- Skip intro **auto**
- Skip credits **button**
- Next episode **after credits** (not countdown)
- Continue Watching **play**
- Artwork cache about **25 MB**, home preload on, enhanced graphics **off**
- Never burn Plex subtitles into the transcode (`burn=0`). Default `subtitleId` 0. Mute native text unless the viewer picked a track.

### Landmines

| Symptom | Cause on this TV |
| --- | --- |
| Audio only, spinner stuck at 99% | Quality cap `1080h` forced HLS / transcode. AVPlay often never fires `onbufferingcomplete`. |
| “Off” still shows captions | Following Plex's selected subtitle burned them into HLS. |
| Play bar never hides while playing | Every clock tick reported buffering-false and reset the hide timer. |
| Play bar hides only when paused | Hide timer required `state === "playing"` and those ticks kept cancelling it. |
| Next episode starts, bar pops back and stays | Start-of-episode buffer/state ticks called `showHud` again and never armed the timer. |
| Episode freezes on the last frame | `STREAM_COMPLETED` leaves native AVPlay **IDLE**. `close()` threw; the leave-guard never ran. |
| Next episode cuts the last minute | Plex **credits** markers fire early. A 10 s “Up Next” countdown treated that as the end. |
| App dies when you press Back | `avplay.stop()` / `close()` during `prepareAsync` native-crashes Tizen 9. |
| Auto skip intro at 0:00 crashes playback | Seeking before the first frame. Gate intro skip on startup-done. |

BIF seek-thumbnails and a 0.5 s / 150 MB artwork budget were also tried. Both made this TV worse. Walked back.

### Next-episode behaviour we ended on

Play the episode through the credits and the real ending (`STREAM_COMPLETED`), then start the next title. Show **Skip Credits** if the viewer wants out. Do not auto-advance when the credits marker appears.

## Dock

Order that felt right on this set: **Vemm, TV Plus, YouTube, AirPlay, Netflix, Disney+, Prime, Spotify, Stremio, Live TV**, then HDMI 1–3 always visible. Official Plex was dropped from the dock in favor of AirPlay. The official Plex app can stay installed.

Weather, if you add it, should be **your** coordinates. Do not copy someone else's city.

## RAM / background hogs

With only Vemm in front, this TV still ran hot if preview / Continue Watching services stayed alive behind the launcher. Stopping known background hogs when Arn0 is home (and not touching AirPlay, Vemm, or Arn0 itself) helped. Do not stop Samsung's AirPlay process to “save RAM.”

## What this repo is not

- Not a fork of Vemm. Vemm is [GPL-3](https://github.com/dragoshont/vemm). If you publish a modified client, publish the source under GPL-3 and say it is a fork.
- Not a signed `.wgt`. Sign on your own TV / DUID.
- Not your certificate, DUID, applist, or LAN addresses. Those do not belong on GitHub or GitLab.

The CU7000-specific playback notes are in [docs/cu7000-playback.md](docs/cu7000-playback.md). The sideload steps are in [docs/install.md](docs/install.md). Safety / restore is in [docs/safety.md](docs/safety.md).

## Credits

- [Vemm](https://github.com/dragoshont/vemm) — Plex-compatible Tizen client
- Arn0 — quiet Tizen home screen used as the dock
- [AeroTizen](https://github.com/helalaou/AeroTizen) — signing / packaging helpers
- [CleanFrame](https://github.com/nthnclrk/CleanFrame) — Home-button approach we did **not** copy onto this Tizen 9 set
- Samsung AVPlay and official Developer Mode

Use at your own risk. Crystal UHD / Tizen 9 is unforgiving, and a bad `close()` will take the whole app with it.
