# Sideload on Tizen 9

Official **Developer Mode** only. The PC and the TV must be on the same LAN.
Retail `sdb` talks to the TV on port **26101**.

## Tools

- Official Samsung `sdb` (Tizen Studio or tizen-sdb). A third-party Python
  handshake failed against the server on this set.
- An author certificate bound to **this** TV's DUID. Reuse it for every update.
- A signer: Tizen CLI, Apps2Samsung, or [AeroTizen](https://github.com/helalaou/AeroTizen).

## App ids used here

| App | Id |
| --- | --- |
| Vemm Preview | `VemmPrevw0.VemmPreview` |
| Arn0 Launcher | `Arn0Lnchr0.Launcher` |
| Home watcher | `Arn0Lnchr0.Watcher` |

Yours can differ. `vd_appinstall` wants the **app id**, not only the filename.

## Sign, push, install

`HOST` is `ip:26101`.

```text
sdb connect HOST
sdb -s HOST push signed.wgt /home/owner/share/tmp/sdk_tools/App.wgt
sdb -s HOST shell 0 vd_appinstall AppId.AppName /home/owner/share/tmp/sdk_tools/App.wgt
sdb -s HOST shell 0 was_execute AppId.AppName
```

If `applist` does not show the new version, `vd_appinstall` did not run.
`sdb install` on this firmware often only copies the file.

## Do not

- Run `sdb shell 0 debug`. It hung Developer Mode on this CU7000.
- Rotate the author certificate and reinstall.
- Expect `sdb` to uninstall store apps. Tizen 9 answered
  `was get app info failed` for several of them.
- Publish a signed `.wgt`, a `.p12`, or a DUID.

## Optional check

Samsung's TV HTTP API on port 8001 can report `running` / `visible` for a
sideloaded app id. Treat that as a convenience check, not a public contract.
