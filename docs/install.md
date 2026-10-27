# Sideload on Tizen 9 (CU7000)

Official **Developer Mode** only. The PC and the TV need to be on the same LAN. Samsung's retail `sdb` talks to the TV on port **26101**.

## Tools

- Samsung `sdb` (Tizen Studio / `tizen-sdb`). AeroTizen's Python SDB handshake failed against the official server on this set — use the real `sdb.exe`.
- An author certificate bound to **this** TV's DUID. Reuse it for every update.
- A signer (Tizen CLI, Apps2Samsung, or [AeroTizen](https://github.com/helalaou/AeroTizen)'s `sign_wgt`).

## Package IDs used here

| App | Package / app id |
| --- | --- |
| Vemm Preview | `VemmPrevw0.VemmPreview` |
| Arn0 Launcher | `Arn0Lnchr0.Launcher` |
| Home watcher | `Arn0Lnchr0.Watcher` |

Yours can differ. `vd_appinstall` wants the **app id**, not just the `.wgt` filename.

## Sign, push, install

Replace `HOST` with the TV's Developer Mode address (`ip:26101`).

```bat
sdb connect HOST
sdb -s HOST push signed.wgt /home/owner/share/tmp/sdk_tools/App.wgt
sdb -s HOST shell 0 vd_appinstall AppId.AppName /home/owner/share/tmp/sdk_tools/App.wgt
sdb -s HOST shell 0 was_execute AppId.AppName
```

`sdb install` on this firmware often only copies the file. If `applist` does not show the new version, you did not run `vd_appinstall`.

## Do not

- `sdb shell 0 debug` — hung Developer Mode on this CU7000 (session died with a wraparound exit code).
- Rotate the author certificate and then “just reinstall.”
- Uninstall store apps through `sdb` and expect success. Tizen 9 answered `was get app info failed` for several of them.
- Publish the signed `.wgt`, the `.p12`, or the DUID.

## Confirm it is running

Samsung's TV REST endpoint on port 8001 reports `running` / `visible` for a sideloaded app id if the set still allows it. Treat that as a convenience check, not an API you should depend on forever.
