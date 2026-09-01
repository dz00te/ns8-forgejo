# Advanced configuration

The module UI exposes the settings most people need: host, TLS, admin
account, and user registration. Forgejo itself has hundreds of
additional `app.ini` settings (mailer, webhooks, repository limits,
2FA enforcement, and more). Exposing all of them in the UI would make
the module unmanageable, so the rest are available as environment
variables you can set by hand.

**Full reference**: every setting below (and everything else Forgejo
supports) is documented in the official
[Configuration Cheat Sheet](https://forgejo.org/docs/latest/admin/config-cheat-sheet/).

## How it works

Forgejo's Docker image translates environment variables into
`app.ini` on every container start. The variable name follows this
pattern:

```
FORGEJO__<section>__<KEY>
```

For example, the `DISABLE_REGISTRATION` key under the `[service]`
section in `app.ini` becomes:

```
FORGEJO__service__DISABLE_REGISTRATION
```

Because `app.ini` is rebuilt from environment variables on every
start, **editing `app.ini` directly does not persist** — any manual
change is overwritten the next time the container restarts. Always
set the environment variable instead.

## Applying a setting

Replace `forgejoN` with your instance name (e.g. `forgejo1`):

```bash
runagent -m forgejoN bash -c "echo 'FORGEJO__section__KEY=value' >> server.env"
runagent -m forgejoN systemctl --user restart forgejo-app.service
```

To check the current value of a variable:

```bash
runagent -m forgejoN grep FORGEJO__section__KEY server.env
```

To remove an override and go back to the module's default, delete the
matching line from `server.env` (e.g. with `sed -i '/^KEY=/d' server.env`)
and restart the service again.

## Common examples

**Enable outgoing mail** (needed for notifications, password reset):

```bash
FORGEJO__mailer__ENABLED=true
FORGEJO__mailer__SMTP_ADDR=smtp.example.org
FORGEJO__mailer__SMTP_PORT=587
FORGEJO__mailer__FROM=forgejo@example.org
FORGEJO__mailer__USER=forgejo@example.org
FORGEJO__mailer__PASSWD=your-smtp-password
```

**Enable Forgejo Actions** (CI/CD, requires a separate runner — not
covered by this module):

```bash
FORGEJO__actions__ENABLED=true
```

**Disable webhooks entirely**:

```bash
FORGEJO__webhook__DISABLE_WEBHOOKS=true
```

**Prevent regular users from creating new repositories**
(admins are unaffected):

```bash
FORGEJO__repository__MAX_CREATION_LIMIT=0
```

**Prevent regular users from creating new organizations**:

```bash
FORGEJO__admin__DISABLE_REGULAR_ORG_CREATION=true
```

**Force everyone to sign in before viewing any page**
(useful for fully private instances):

```bash
FORGEJO__service__REQUIRE_SIGNIN_VIEW=true
```

## A note on support

These variables are set at your own risk: they are not validated by
the module, and an invalid value can prevent Forgejo from starting.
If that happens, remove the offending line from `server.env` and
restart again. Back up `server.env` before experimenting.
