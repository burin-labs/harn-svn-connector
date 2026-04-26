# harn-svn-connector

Pure-Harn Subversion connector: post-commit hook normalization and polling revision scanner surface.

This package implements the Harn Connector interface contract v1 for `svn`.
It normalizes inbound webhook payloads to the tagged `NormalizeResult` envelope,
verifies provider-specific webhook signatures, and exposes outbound raw API helpers
plus common PR/comment/status method aliases.

## Install

```sh
harn add github.com/burin-labs/harn-svn-connector@v0.1.0
```

Until a version is tagged, depend on a path checkout:

```toml
[dependencies]
harn-svn-connector = { path = "../harn-svn-connector" }
```

## Webhook verification

The connector accepts unsigned events only when no verification material is
configured. Configure `signing_secret` for HMAC-based providers or `public_key`
for SourceHut Ed25519 verification.

## Authentication

Outbound calls use repository URL plus host-managed SVN credentials for polling. Pass `access_token`, `token`,
`personal_access_token`, or `app_password` in the call args, or set the
`SVN_TOKEN` environment variable.

## Development

```sh
harn check src/lib.harn
harn fmt --check src/lib.harn tests/*.harn
for test in tests/*.harn; do harn run "$test" || exit 1; done
```

## License

Dual-licensed under MIT and Apache-2.0.
