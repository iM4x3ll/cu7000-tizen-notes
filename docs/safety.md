# Safety and restore

Stay inside Samsung **Developer Mode** and signed sideloads.

## Do not

- Enable SSH or open port 22 on the TV.
- Use root, archive-root, or Home-button exploits.
- Weaken someone else's native Home helper to run it here.
- Commit certificates, DUID files, signed `.wgt` packages, applists, or LAN IPs.
- Uninstall AirPlay, iPhone Remote, TV Plus, or Samsung Home to save RAM.

## Restore

Uninstall **only** packages you sideloaded, then confirm they are gone from
`applist`. If you turned **Start with Smart Hub Home** off, turn it back on
for stock boot. AirPlay, TV Plus, and official Plex should still be there.

If a sideloaded app will not open after an update, the usual cause is an
**author certificate mismatch**. Sign with the same certificate as the
previous build. Do not uninstall first unless you are willing to lose that
app's local settings.

## Background processes

Stopping random Samsung services from a launcher is how AirPlay disappears.
If you stop preview services when the custom home is in front, name them and
never include AirPlay, Vemm, or the launcher.
