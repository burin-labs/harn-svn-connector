# CLAUDE.md - harn-svn-connector

Pure-Harn Subversion connector package for post-commit hooks and polling revision scans.

Shared Harn connector authoring rules live in the canonical guide:

- https://github.com/burin-labs/harn/blob/main/docs/src/connectors/authoring.md

Keep this file limited to provider-specific notes and local hazards. Add shared connector guidance
to the Harn guide first.

## Provider Notes

- This provider supports both webhook and poll trigger kinds; poll fixtures should enable poll-tick
  execution in CI.
- Webhook event names use `x-svn-event`; revision/delivery material comes from `x-svn-revision` or
  `x-request-id`.
- Webhook signing is optional HMAC via `x-harn-svn-signature` when a signing secret is configured.
- Outbound/polling behavior depends on repository URL plus host-managed SVN credentials; avoid
  assuming a Git-style PR surface exists for every SVN deployment.
