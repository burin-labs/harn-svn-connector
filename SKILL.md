# SKILL: harn-svn-connector

Use `harn-svn-connector` when wiring Harn triggers or outbound helpers for Subversion.

## What you get

- Provider id: `svn`
- Trigger kinds: `webhook, poll`
- Supported events: `commit, branch, tag, property_change`
- Webhook verification: `optional_hmac`
- Outbound helpers: `api.request`, `pull_requests.comment`, `pull_requests.update`, `issues.comment`, `commit_status.set`, `repository_file.get`

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

For SourceHut, use a configured `public_key` secret instead of `signing_secret`.
For SVN polling, add a `poll` trigger and run `harn connector check . --provider svn --run-poll-tick`.
