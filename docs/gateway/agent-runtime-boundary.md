---
summary: "Operational policy for host-level coding-agent CLIs — temporary rescue tools only"
read_when:
  - Running or debugging host-level CLI agents alongside the gateway
  - Onboarding or maintaining host-level agent access
title: "Agent runtime boundary"
---

<Warning>
  **Host-level coding-agent CLIs are temporary rescue tools only.**
  Normal agent work must run inside the managed Gateway runtime, not from a
  long-lived authenticated host CLI.
</Warning>

## Rationale

The Gateway runtime provides a controlled execution environment with
built-in session management, sandboxing, credential isolation, and audit
logging. Host-level CLI agents (Codex, host-shell) bypass these controls:

- They run with the full privileges of the host user
- They have unrestricted filesystem and network access
- They inherit all environment variables, including secrets
- Their sessions are not managed or audited by the Gateway
- They leave no trace in Gateway logs

## When host-level access is acceptable

Host-level coding-agent CLIs may be used **temporarily** for:

1. **Bootstrap and recovery** — initial gateway setup, troubleshooting a
   broken Gateway runtime, or performing a one-time migration that the
   Gateway cannot execute
2. **Emergency diagnostics** — reading system-level logs, inspecting
   process state, or debugging a Gateway crash
3. **One-shot operations** — a single command that is impractical to run
   through the Gateway (e.g., `systemctl`, `docker inspect`)

Host-level access is **not** acceptable for:

- Routine agent sessions or daily work
- Long-running autonomous tasks
- Operations that involve user data, credentials, or external services
- Any task that the Gateway runtime can perform

## Procedure for enabling host-level access

1. **Document the reason** — create a tracking issue or log entry stating
   why host-level access is needed, who requested it, and the expected
   removal window
2. **Archive session state** — before disabling live access, archive any
   active session threads and state. Exclude raw logs and authentication
   material from the archive
3. **Remove or move live auth material** — do not leave credentials or
   tokens accessible to the host-level CLI
4. **Replace entrypoints** — replace host CLI entrypoints (e.g. `codex`,
   `codex-host-real`) with a fail-closed wrapper that refuses to run, or
   remove them from `PATH`
5. **Verify Gateway runtime** — confirm that the managed Gateway runtime
   can still list and start agents normally
6. **Set a removal deadline** — the host-level access must be removed
   within a defined window (hours, not days)

## Shutdown checklist

When removing host-level access:

- [ ] Archive session state (`tar.gz` + SHA-256)
- [ ] Strip auth material from the archive
- [ ] Remove or fail-close host entrypoints
- [ ] Clear any cached credentials from the shell environment
- [ ] Verify `codex --version` (or equivalent) exits non-zero
- [ ] Verify Gateway agent list works: `openclaw agents`
- [ ] Log the cleanup completion with a timestamp

## Cleanup policy

- **Orphaned agent processes**: run daily cleanup for clearly orphaned
  agent processes
- **Detached terminal sessions**: use more conservative cleanup, as they
  may contain intentional long-running work
- **Auth material**: never commit credentials, tokens, or session state
  to Git repositories
- **Archives**: store archives outside the repository, with documented
  SHA-256 checksums

## Related

- [Security overview](/gateway/security)
- [Sandboxing](/gateway/sandboxing)
- [Gateway runbook](/gateway)
