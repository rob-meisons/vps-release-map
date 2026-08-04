# vps-release-map

Release map for the VPS fleet managed by
[web-installer](https://github.com/rob-meisons/web-installer).

This repository holds deployment decisions only — which immutable Git tag each
release-pinned environment should be serving. It contains no application code
and no secrets. Keeping it separate from the app repos gives those decisions
their own history and access control, and lets a promotion or rollback happen
without touching application code.

## Layout

A single [`releases.yaml`](releases.yaml) covers the whole fleet, keyed by
`{app}.{env}`:

```yaml
weather:
  live: v1.0.0
```

Only environments configured with `sync: release` in `sites.yaml` appear here.
Environments left in the default `sync: branch` mode follow a branch tip and are
not listed.

## Promoting or rolling back

1. Tag the release in the app's own repository and push the tag.
2. Edit the environment's value in `releases.yaml`.
3. Commit and push here.
4. On the VPS, run `wi update <app> <env>` — or wait for that environment's
   daily update timer.

The VPS clones this repo read-only and checks out the configured branch on every
update. An update is a no-op when the selected tag already resolves to the
commit that is checked out. Rollback is symmetric: release sync will move an
environment backward when the tag you select is older than what it is serving.

If a tag named here does not exist in the app repo, the update run fails and the
environment keeps serving its current checkout.

## Consumed by

`sites.yaml` in web-installer, via a per-environment `release:` block:

```yaml
release:
  source: repo
  repo: <clone URL of this repository>
  branch: main
  file: releases.yaml
  key: "{app}.{env}"
```
