# harn-svn-connector

Pure-Harn Subversion connector: post-commit hook normalization and polling
revision scanner surface.

This package implements the Harn Connector interface contract v1 for `svn`.
It normalizes inbound webhook payloads to the tagged `NormalizeResult` envelope,
verifies provider-specific webhook signatures, and exposes a small read-only
outbound API surface.

## Inbound-first by design

Subversion has no pull-request, issue, or commit-status write API, so this
connector is **inbound-first**: its event surface is the post-commit hook
(`normalize_inbound`) plus the polling revision scanner (`poll_tick`). The
outbound `call` surface is intentionally minimal and read-only. There are no
PR/issue/commit-status methods — those would be misleading for SVN.

## Install

```sh
harn add github.com/burin-labs/harn-svn-connector@v0.1.0
```

Use a path checkout for unreleased `main` or local multi-repo development:

```toml
[dependencies]
harn-svn-connector = { path = "../harn-svn-connector" }
```

## Webhook verification

Subversion hook deliveries must be signed. Configure `signing_secret` or
`SVN_SIGNING_SECRET`; the connector verifies the `x-harn-svn-signature`
HMAC-SHA256 header against the raw request body and rejects requests with no
configured secret, missing binding id, missing signature, or invalid signature.

## Authentication

Outbound calls use the repository URL plus host-managed SVN credentials. SVN
prefers HTTP Basic when `username` + `password` (or `$SVN_USERNAME` /
`$SVN_PASSWORD`) are present; an explicit `access_token` / `token` is attached as
Bearer. Authorization is never attached over plaintext `http://`.

## Outbound read helpers

All read-only, over the configured SVN HTTP (DAV) endpoint; pass `path` to
override the exact request path:

- `call("repository.info", {repo_path?})` — repository root / youngest revision
  (`svn info` semantics).
- `call("revision.get", {revision, repo_path?})` — read a single revision
  (`svn log -r <rev>` semantics).
- `call("revision.log", {from_revision?, to_revision?, limit?, repo_path?})` —
  read a revision log range (`svn log` semantics).
- `call("api.request", ...)` / `call("paginate", ...)` — generic escape hatches.

## Development

```sh
harn check src/lib.harn
harn fmt --check src/lib.harn tests/*.harn
harn connector test . --provider svn --run-poll-tick
```

## License

Dual-licensed under MIT and Apache-2.0.
