# Image pin changelog

Records the upstream LibreBooking image tag each tenant runs, plus any active workarounds layered on top.

## Current pin

- **Upstream image**: `librebooking/librebooking:5.0.2`
- **Workaround active**: yes — Custom Start Command on each Railway service removes `mpm_event` and `mpm_worker` symlinks from `/etc/apache2/mods-enabled/` at runtime
- **Tracking**: file an issue at https://github.com/LibreBooking/docker/issues and link it here

## The MPM bug

Upstream `librebooking/librebooking` images rebuilt between 2026-04-30 and 2026-05-06 ship with **both** `mpm_event` and `mpm_prefork` Apache modules symlinked in `/etc/apache2/mods-enabled/`. This causes Apache to refuse to start with:

```
AH00534: apache2: Configuration error: More than one MPM loaded.
```

`mod_php` (which LibreBooking uses) is not thread-safe and requires `mpm_prefork`. We must disable `mpm_event`.

Build-time fixes (via Dockerfile `RUN` to `rm` the symlinks) did NOT reliably persist through Railway's build/deploy pipeline. The fix that works is a runtime cleanup via Railway's Custom Start Command:

```bash
bash -c 'rm -fv /etc/apache2/mods-enabled/mpm_event.* /etc/apache2/mods-enabled/mpm_worker.* 2>&1 || true; exec /usr/local/bin/entrypoint.sh apache2-foreground'
```

The `Dockerfile` itself just sets `USER root` so the runtime cleanup has permission to delete the symlinks.

## History

| Date       | Tenant         | Tag    | Workaround | Notes |
|------------|----------------|--------|------------|-------|
| 2026-05-06 | tenant-pilot-01 | 4.3.0  | Runtime MPM fix via Custom Start Command | Initial pilot deploy. |
| 2026-05-06 | tenant-pilot-01 | 5.0.2  | Runtime MPM fix via Custom Start Command | Bumped to latest stable; same MPM workaround applies. |

## Upgrade procedure

1. Pick the new upstream tag (Docker Hub: https://hub.docker.com/r/librebooking/librebooking/tags)
2. Edit `deploy/Dockerfile` → bump the `FROM` line
3. **Test the new tag without the Custom Start Command on a scratch Railway project first.** If Apache starts cleanly without the workaround, the upstream MPM bug is fixed:
   - Remove the Custom Start Command from all tenants
   - Update `deploy/Dockerfile` to remove the `USER root` line (no longer needed)
   - Mark workaround inactive in this CHANGELOG
4. If the bug persists in the new tag, keep the workaround
5. Test on `tenant-pilot-01` first (smoke test booking flow + redeploy persistence), then roll forward to other tenants
6. Append a row to the History table above
