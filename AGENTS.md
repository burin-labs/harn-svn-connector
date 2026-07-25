# AGENTS.md

Pure-Harn Subversion connector package for post-commit hooks and polling
revision scans.

Shared connector authoring rules live in the Harn guide:

- [Connector authoring guide](https://github.com/burin-labs/harn/blob/main/docs/src/connectors/authoring.md)

Put shared connector guidance in the Harn guide and keep only
provider-specific notes and local hazards here.

`CLAUDE.md` points here. Edit `AGENTS.md` only.

## Provider notes

- This provider supports webhook and poll triggers. Poll fixtures should enable
  poll-tick execution in CI.
- Webhook event names use `x-svn-event`; revision or delivery material comes
  from `x-svn-revision` or `x-request-id`.
- Webhook signing uses `x-harn-svn-signature` HMAC. Each request must identify
  a binding with a signing secret. Both `sha256=<hex>` and bare `<hex>` are
  accepted by `std/connectors/shared::verify_hmac_signature`.
- Outbound calls return `{status, headers, body, rate_limit}`. The parsed JSON
  body lives at `response.body`, and `x-ratelimit-*` plus `retry-after`
  headers are surfaced in `response.rate_limit`.
- Outbound calls share a process-local token bucket through
  `std/connectors/shared`. It is keyed by `args.rate_limit.scope_key`, then
  `args.repository_url`, then `args.binding_id`. Set
  `rate_limit = {capacity, refill_tokens, refill_interval_ms, scope_key?}`
  per call or through the binding or `init` context; omit it to disable the
  bucket.
- The `paginate` method wraps `std/connectors/shared::paginate_cursor` for
  SVN-style cursor APIs. Configure `cursor_param` (default `cursor`),
  `cursor_path` (default `/next_cursor`), `items_path`, `max_pages`, and
  `initial_cursor` for the target log or changelist endpoint.
- Outbound and polling behavior depends on the repository URL plus
  host-managed SVN credentials. Do not assume a Git-style PR surface.
