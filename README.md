# CU7000 / Tizen 9 living-room notes

Working notes from one Samsung **UN65CU7000FXZA** (Tizen 9, armv7): a quiet
sideloaded home screen, a Plex client that Direct Plays, and the stock
features that are worth keeping.

This is not a product, a root kit, or a fork of anyone else's client.
**Tested on one TV.** Treat every other model and firmware as unproven.

- GitHub: https://github.com/iM4x3ll/cu7000-tizen-notes
- GitLab: https://gitlab.com/Maxwell333/cu7000-tizen-notes
- Vemm report: https://github.com/dragoshont/vemm/issues/8

## Kept

- AirPlay 2
- iPhone Remote
- Samsung TV Plus
- YouTube
- Official Plex (installed, not on the dock)

## Used

| Piece | Role |
| --- | --- |
| [Vemm](https://github.com/dragoshont/vemm) Preview (`VemmPrevw0.VemmPreview`) | Plex-compatible AVPlay client (GPL-3) |
| Arn0 Launcher (`Arn0Lnchr0.Launcher`) plus Home watcher | Sideloaded dock; Home returns here |
| Official Developer Mode | Signed `.wgt` sideload only |

Samsung Home stays installed. **Autorun Last App** does not treat a sideloaded
app as the last app on this firmware, so the watcher still matters.

## Refused

- SSH / port 22
- Root, archive-root, or Home-button exploits
- `sdb shell 0 debug` (it hung Developer Mode on this set)

A guide that roots a 2023 Samsung to replace Home is the wrong guide for this TV.

## Sideload

Retail `sdb` on Tizen 9 is reliable for `applist`, `getduid`, and install /
uninstall. A plain `sdb install` often **only copies** the file.

1. Enable Developer Mode and bind the PC.
2. Sign with the **same** author certificate on every update.
3. `sdb push` the signed `.wgt` to `/home/owner/share/tmp/sdk_tools/`.
4. Install with `sdb shell 0 vd_appinstall <appid> <remote-path>`.
5. Launch with `sdb shell 0 was_execute <appid>` or from the app list.

Do not rotate the author certificate. A mismatch can force an uninstall and
wipe that app's local settings. Store apps did not uninstall cleanly through
`sdb` here; leave them, or remove them from the TV menus.

Details: [docs/install.md](docs/install.md).

## Boot

- If the TV still wakes into Smart Hub, turn **Start with Smart Hub Home**
  off and **Autorun Samsung TV Plus** off on the TV.
- The Home watcher brings the launcher back after Home. About 1.5 s was enough.
- Restore by uninstalling **only** packages you sideloaded. Do not remove
  Samsung Home, AirPlay, or TV Plus.

Details: [docs/safety.md](docs/safety.md).

## Vemm on this set

Vemm was the best AVPlay Plex client compared here. Stock defaults are not
CU7000-safe.

Stable settings:

- Quality **original**, Direct Play **auto**, min buffer **1 s**
- Skip intro **auto**, skip credits **button**
- Next episode **after credits** (not countdown)
- Continue Watching **play**
- Artwork about **25 MB**, home preload on, enhanced graphics **off**
- Never burn Plex subtitles (`burn=0`). Default `subtitleId` 0 unless the
  viewer picked a track in this client

| Symptom | Cause |
| --- | --- |
| Audio only, spinner at 99% | `1080h` forced HLS. AVPlay often never fires `onbufferingcomplete`. |
| Off still shows captions | Following Plex's selected subtitle burned them into HLS. |
| Play bar never hides | Clock ticks reported buffering-false and reset the hide timer. |
| Bar hides only when paused | Hide timer requires `state === "playing"`. |
| Next episode, bar stays up | Start-of-episode ticks called `showHud` and never armed the timer. |
| Freeze on the last frame | `STREAM_COMPLETED` leaves AVPlay **IDLE**; `close()` threw. |
| Last minute cut off | Plex credits markers fire early; a 10 s countdown treated that as the end. |
| Back kills the app | `stop()` / `close()` during `prepareAsync` native-crashes Tizen 9. |
| Intro skip at 0:00 crashes | Seek before the first frame. |

A 0.5 s buffer, a 150 MB artwork budget, and BIF seek-thumbs all made this TV
worse.

Play through the credits and the real ending, then start the next title. Keep
**Skip Credits** for a manual jump.

Details: [docs/cu7000-playback.md](docs/cu7000-playback.md).

## Dock

Vemm, TV Plus, YouTube, AirPlay, Netflix, Disney+, Prime, Spotify, Stremio,
Live TV, then HDMI 1–3 always shown. Official Plex can stay installed off the
dock.

Do not stop Samsung's AirPlay process to save RAM. Preview / Continue Watching
services behind the launcher were the hogs here.

## This repo is not

- A Vemm fork. Modified Vemm stays [GPL-3](https://github.com/dragoshont/vemm)
  and should say it is a fork.
- A signed `.wgt`. Sign on your own TV.
- A place for certificates, DUIDs, applists, or LAN addresses.

## Credits

- [Vemm](https://github.com/dragoshont/vemm)
- Arn0 (sideloaded dock)
- [AeroTizen](https://github.com/helalaou/AeroTizen) (signing helpers)
- [CleanFrame](https://github.com/nthnclrk/CleanFrame) (Home approach **not**
  used on this Tizen 9 set)
- Samsung AVPlay and official Developer Mode

Use at your own risk.
