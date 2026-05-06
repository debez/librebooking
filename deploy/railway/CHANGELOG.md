# Image pin changelog

Records the upstream LibreBooking image tag each tenant runs, plus any active workarounds layered on top.

## Current pin

- **Upstream image**: `librebooking/librebooking:4.3.0`
- **Workaround active**: yes — `deploy/Dockerfile` disables `mpm_event` / `mpm_worker` and forces `mpm_prefork` to work around upstream MPM bug
- **Tracking**: file an issue at https://github.com/LibreBooking/docker/issues and link it here

## History

| Date       | Tenant         | Tag    | Dockerfile workaround | Notes |
|------------|----------------|--------|-----------------------|-------|
| 2026-05-06 | tenant-pilot-01 | 4.3.0  | yes (MPM fix)         | Initial pilot deploy. 5.0.2 also broken with same MPM error. |

## Upgrade procedure

1. Pick the new upstream tag (Docker Hub: https://hub.docker.com/r/librebooking/librebooking/tags)
2. Edit `deploy/Dockerfile` → bump the `FROM` line
3. If the upstream MPM bug is fixed in the new tag: delete `deploy/Dockerfile` entirely and switch Railway service Source back to "Image" → `librebooking/librebooking:<new-tag>`
4. Test on `tenant-pilot-01` first (smoke test booking flow + redeploy persistence), then roll forward to other tenants
5. Append a row to the History table above
