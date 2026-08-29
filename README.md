# homelab-policy

A cfbs `module-repo` providing three independently addable modules for
reuse across cfbs-built policy sets (e.g.
`orhagen-web-cfengine-policy-cfbs`):

- **`homelab-policy`** (subdirectory `homelab-policy/`) — role bundles:
  `role_common`, `role_proxmox`, `role_k3s_server`, `role_database`,
  `role_fileserver`, `role_cfengine_hub`, `role_vm`, wired into
  `bundle agent homelab_policy_roles` (tagged `autorun`) via
  `homelab-policy/autorun/roles.cf`, gated on the same host-identity
  classes as the source repo (`orhagen_no_server`,
  `postgresql_server_turnkey`, `fileserver_turnkey`, `cfengine_hub`,
  `pm1`).
- **`install-tailscale`** (subdirectory `install-tailscale/`) — Tailscale
  install/bootstrap (`bundle agent tailscale`, tagged `autorun`). Requires
  a Tailscale auth key published via a shared CFEngine file. Depends on
  `site-classes` (in addition to `autorun`), since `tailscale.cf` gates
  its container-networking fix on the `is_container` class.
- **`site-classes`** (subdirectory `site-classes/`) — `common_classes.cf`
  (`bundle common site_classes`, defining `is_lxc`/`is_docker`/
  `is_container`), copied as-is straight into `services/autorun/` so it
  loads and runs on its own — `bundle common` bundles evaluate
  automatically once loaded, no autorun tag or explicit `inputs`
  registration needed. Kept separate from `homelab-policy` so
  `install-tailscale` can depend on just this small module instead of
  pulling in all seven role bundles.

Each module declares `"dependencies": ["autorun"]`, since
`services_autorun` is off by default in stock masterfiles and needs the
`autorun` cfbs module (or an equivalent `def.json` class) to make files
copied into `services/autorun/` actually get picked up by the Masterfiles
Policy Framework.

## Requirements

- Assumes the consuming policy set already provides the standard CFEngine
  stdlib bodies (`mog`, `if_repaired`, `if_ok`, `if_elapsed`, `in_shell`,
  etc.) — normally pulled in via the `masterfiles` build entry, same as in
  `orhagen-web-cfengine-policy-cfbs/cfbs.json`.
- Host-identity classes (`orhagen_no_server`, `pm1`, `Hr00`/`Hr03` time
  classes, etc.) are expected to already be defined by the consuming
  policy set's own inventory/class-setting policy.

## Using it

From a cfbs project (adjust the path to wherever this repo is checked
out relative to the project):

```
cfbs add ../orhagen-web-cfengine-policy/modules/homelab-policy//homelab-policy
cfbs add ../orhagen-web-cfengine-policy/modules/homelab-policy//install-tailscale
cfbs build
```

`homelab-policy` copies files under `services/homelab-policy/roles/` and
`services/autorun/homelab-policy-roles.cf`; `install-tailscale` copies
`services/autorun/homelab-policy-tailscale.cf`; `site-classes` (pulled in
automatically by `install-tailscale`) copies
`services/autorun/homelab-policy-common-classes.cf`. All are namespaced
to avoid colliding with a project's own `services/roles/` or
`services/autorun/roles.cf`.

## Note

`homelab-policy/roles/common.cf` contains real host IPs and an SSH
public key (`hosts_entries` / `authorized_keys_entries`). This repo is
meant for Ole's own infrastructure, not for public distribution as-is.
