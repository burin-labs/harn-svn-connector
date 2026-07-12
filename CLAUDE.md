# CLAUDE.md - harn-svn-connector

Pure-Harn Subversion connector package for post-commit hooks and polling revision scans.

Shared Harn connector authoring rules live in the canonical guide:

- https://github.com/burin-labs/harn/blob/main/docs/src/connectors/authoring.md

Keep this file limited to provider-specific notes and local hazards. Add shared connector guidance
to the Harn guide first.

## Provider notes

- This provider supports both webhook and poll trigger kinds; poll fixtures should enable poll-tick
  execution in CI.
- Webhook event names use `x-svn-event`; revision/delivery material comes from `x-svn-revision` or
  `x-request-id`.
- Webhook signing is optional HMAC via `x-harn-svn-signature` when a signing secret is configured;
  both `sha256=<hex>` and bare `<hex>` are accepted, verified through
  `std/connectors/shared::verify_hmac_signature`.
- Outbound calls return `{status, headers, body, rate_limit}`; the parsed JSON body lives at
  `response.body` and `x-ratelimit-*` plus `retry-after` headers are surfaced in `response.rate_limit`.
- Outbound calls share a process-local token bucket via `std/connectors/shared` keyed by
  `args.rate_limit.scope_key`, falling back to `args.repository_url`, then `args.binding_id`.
  Pass `rate_limit = {capacity, refill_tokens, refill_interval_ms, scope_key?}` per-call or via the
  binding / `init` ctx; absence disables the bucket.
- The `paginate` outbound method wraps `std/connectors/shared::paginate_cursor` for SVN-style
  cursor APIs; configure `cursor_param` (default `cursor`), `cursor_path` (default `/next_cursor`),
  `items_path`, `max_pages`, and `initial_cursor` to fit the target host's log/changelist endpoint.
- Outbound/polling behavior depends on repository URL plus host-managed SVN credentials; avoid
  assuming a Git-style PR surface exists for every SVN deployment.
