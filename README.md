# ns8-forgejo

[Forgejo](https://forgejo.org) is a self-hosted, lightweight software
forge for Git hosting, code review, and more. This module packages
Forgejo for [NS8](https://github.com/NethServer/ns8-core).

## Features

- SQLite database (no external database required)
- HTTP access behind Traefik, with optional Let's Encrypt certificate
- Public SSH access for `git clone`/`git push`, with its own firewall
  port
- Automatic admin account creation on first configuration
- User self-registration disabled by default (only the admin can
  create accounts)

## Installation

```bash
add-module ghcr.io/dz00te/forgejo:latest 1
```

## Configuration

Configuration can be done from the module's Settings page in the NS8
UI, or via `api-cli`:

```bash
api-cli run configure-module --agent module/forgejo1 --data - <<EOF
{
  "host": "forgejo.example.org",
  "http2https": true,
  "lets_encrypt": true,
  "admin_password": "a-strong-password",
  "admin_username": "forgejo-admin",
  "admin_email": "admin@example.org",
  "allow_registration": false
}
EOF
```

Only `host`, `http2https`, `lets_encrypt`, and `admin_password` are
required. `admin_username` defaults to `forgejo-admin` if omitted
(note: `admin` is a reserved username in Forgejo and cannot be used).
`admin_email` defaults to `admin@<host>`.

## Using Git over SSH

The module reserves a public TCP port for SSH access, separate from
the HTTP port behind Traefik. After configuration, the SSH clone URL
is shown on your repository pages inside Forgejo (e.g.
`ssh://git@forgejo.example.org:20004/user/repo.git`).

## Advanced configuration

The settings above cover the common cases. Forgejo has hundreds of
additional `app.ini` options (mail, webhooks, Actions, repository
limits, and more) that can be set manually as environment variables.
See [Advanced configuration](docs/ADVANCED_CONFIG.md) for details and
examples.

## Uninstallation

```bash
remove-module --no-preserve forgejo1
```

## License

GPL-3.0-or-later

The Forgejo logo used in this module's UI is courtesy of the
[Forgejo project](https://codeberg.org/forgejo/governance/src/branch/main/branding),
licensed under CC-BY-SA-4.0.
