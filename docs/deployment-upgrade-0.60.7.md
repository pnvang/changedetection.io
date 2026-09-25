# Docker deployment: 0.60.7

The Compose service uses the official 0.60.7 image, pinned to its multi-platform
manifest digest. This updates the container runtime; it does not update the
Python source checkout or a pip installation. Keep existing local environment,
port, network and volume settings when applying this image change.

## Before upgrading from 0.55.7

1. Pull the pinned image before interrupting the running service:
   `docker compose pull changedetection`.
2. Record the running container's image ID, mounts and effective Compose
   configuration. Retain the old image under a local rollback tag.
3. Stop only this service: `docker compose stop changedetection`.
4. Copy **all** of `/datastore` from the stopped container to a private backup
   directory outside the volume and Git repository. For the default container
   name, use `docker cp changedetection:/datastore /absolute/backup/directory/`.
   Archive the copy, record its SHA-256 checksum and verify it can be extracted.
   The backup includes notification credentials and must remain private.
5. Start the service with `docker compose up -d --no-deps changedetection` from
   the original deployment directory, preserving its Compose project name.

The datastore migrates automatically from schema 32 to 33. Migration 33 changes
restock price fields. The application's automatic migration backup does not
include historical snapshots and is not a substitute for the full backup.

## Verify and roll back

Confirm runtime version 0.60.7, schema 33, unchanged watch/group identities and
preserved history. Check the watch list, edit and history pages, representative
scheduled fetches, notification configuration and the deployed public URL.
Check logs for new migration or fetch failures. Watches combining `source:` URLs
with multiple XPath matches need special attention due to
[upstream issue 4477](https://github.com/dgtlmoon/changedetection.io/issues/4477).

If rollback is needed, stop this service, restore the full pre-upgrade backup
into a **fresh** Docker volume, and configure this service to use that volume
and the retained old image. Preserve the upgraded volume for diagnosis. Merely
downgrading the image does not undo schema 33 or restore the original price data.

[Release notes](https://github.com/dgtlmoon/changedetection.io/releases/tag/0.60.7)
and [migration source](https://github.com/dgtlmoon/changedetection.io/blob/0.60.7/changedetectionio/store/updates.py).
