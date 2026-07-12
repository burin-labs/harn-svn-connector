# SKILL: harn-svn-connector

Use `harn-svn-connector` when wiring Harn triggers or outbound helpers for Subversion.

## What you get

- Provider id: `svn`
- Trigger kinds: `webhook, poll`
- Supported events: `commit, branch, tag, property_change`
- Webhook verification: `hmac_sha256`
- Outbound helpers: `repository.info`, `revision.get`, `revision.log`,
  `api.request`, and `paginate`

## Trigger recipe

```toml
[[triggers]]
id = "svn-events"
kind = "webhook"
provider = "svn"
match = { path = "/hooks/svn", events = ["commit"] }
handler = "handlers::on_svn_event"
secrets = { signing_secret = "svn/signing-secret" }
```

Webhook bindings require a valid HMAC signature. For revision polling, use a
`poll` trigger; `poll_tick` returns the next revision batch from its cursor.

Validate webhook and polling behavior with
`harn connector test . --provider svn --run-poll-tick`.
