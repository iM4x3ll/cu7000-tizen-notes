# Safety and restore

This setup stays inside Samsung **Developer Mode** and signed sideloads. That is already enough rope.

## Do not

- Enable SSH or open port 22 on the TV.
- Use archive-root, Vetro, or CVE writeups (including SVE-2025-50109) to remap Home.
- Weaken someone else's native Home helper “just to see.”
- Commit certificates, DUID files, signed `.wgt` packages, applist dumps, or LAN IPs.
- Uninstall AirPlay, the iPhone Remote stack, TV Plus, or Samsung Home to save RAM.

## Restore

Keep a script that uninstalls **only** packages you sideloaded, for example:

```
vd_appuninstall VemmPrevw0
vd_appuninstall DclHome001
```

Then:

1. Confirm `applist` no longer shows those ids.
2. If you turned **Start with Smart Hub Home** off, turn it back on if you want stock boot.
3. AirPlay / TV Plus / official Plex should still be there.

If the launcher or Vemm will not open after an update, the usual cause is an **author certificate mismatch**, not a corrupt TV. Install the same cert you signed the previous build with. Do not uninstall first unless you are willing to lose that app's local settings.

## Background processes

Stopping random Samsung services from a launcher is how you lose AirPlay. If you stop preview / Continue Watching hogs when the custom home is in front, name them, back them up, and never include AirPlay, Vemm, or the launcher itself.
