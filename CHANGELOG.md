# Changelog

## Unreleased

- Removed the vestigial gitea-shaped outbound `call` stubs that made no sense for Subversion
  (forge parity, C5): `pull_requests.comment`, `pull_requests.update`, `issues.comment`, and
  `commit_status.set`, plus the `repos/<owner>/<repo>/...` path builders and the now-unused
  `__config_keys` / `__body_without` helpers. Subversion has no pull-request, issue, or
  commit-status write API, so these were misleading. They now return `method_not_found`.
- Reduced the outbound surface to SVN-appropriate read-only operations over the configured SVN
  HTTP (DAV) endpoint:
  - `repository.info` — read repository root / youngest revision (`svn info` semantics).
  - `revision.get` — read a single revision (`svn log -r <rev>` / `svn info -r <rev>` semantics);
    requires `revision`.
  - `revision.log` — read a revision log range (`svn log` semantics); optional `from_revision`
    / `to_revision` / `limit`.
  - The generic `api.request` / `paginate` escape hatches are unchanged for anything else the
    server exposes.
- **SVN is an inbound-first connector.** Its event surface is the post-commit hook
  (`normalize_inbound`) plus the polling revision scanner (`poll_tick`); both are unchanged. The
  outbound surface is intentionally minimal and read-only because Subversion exposes no
  meaningful outbound write API in this connector's scope.
- Security sweep 2026-05-23 (hardening, fail-closed defaults):
  - **F1 (CRITICAL) SSRF:** `__api_url` rejects absolute-URL `path` arguments so the
    configured `api_base_url` (and any attached `Authorization` header) cannot be redirected
    to attacker-chosen hosts.
  - **F2 (HIGH) https default + plaintext refusal:** `DEFAULT_API_BASE_URL` is now
    `https://svn.example.com`. `__api_request` refuses to attach an `Authorization` header
    when the target URL is `http://`, returning `insecure_transport` instead of leaking
    credentials over plaintext.
  - **F3 (HIGH) fail-closed:** webhook delivery is rejected with `401 missing_signature`
    when no signing secret is configured for the binding.
  - **F5 (HIGH) trusted secret resolution:** `__signing_secret` no longer reads `raw.*`
    fields. Secrets resolve from orchestrator `ctx`, the per-binding state seeded via
    `activate()`, or `$SVN_SIGNING_SECRET` only.
  - **F6 (HIGH) body source:** HMAC runs over `body_text` when present, else over the
    base64-decoded bytes of `body_base64`. Absent body now rejects `400 missing_body`.
  - **F7 (MEDIUM) exact event match:** `__supports_event` matches exact names or
    fully-qualified subkinds; bare prefix matches are gone.
  - **F8 (MEDIUM) require binding_id:** rejects `400 missing_binding_id` when inbound
    requests have no binding pointer.
  - **F9 (MEDIUM) dedupe key:** derived from the host-set `x-svn-revision` / `x-request-id`
    headers only; falls back to `sha256(body_base64 | received_at | binding_id)`.
  - **F11 (MEDIUM) per-provider auth:** SVN uses HTTP Basic when `username` (or
    `$SVN_USERNAME`) and `password` (or `$SVN_PASSWORD`) are present; Bearer is attached
    only when the caller passes an explicit `access_token` / `token`. `app_password` and
    `personal_access_token` are no longer auto-aliased into Bearer.
  - **F13 (LOW) shutdown gating:** documented `@host_only` semantics for `shutdown()`.
  - **F14 (LOW) verify before parse:** signature verification runs ahead of `json_parse`.
  - Deferred:
    - **F10** SVN polling stub is still non-functional. Resolving needs an implement-or-
      remove decision; the next maintainer should either wire `svn log` + XML parsing or
      drop `poll` from `kinds()` and return `__err("not_implemented")` from `poll_tick`.
    - **F12** Outbound dispatch consolidation.
    - **F15** Bump-harn workflow scoping.

## v0.1.0

- Initial pre-alpha Subversion connector package implementing Harn Connector contract v1.
