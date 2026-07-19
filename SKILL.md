---
name: grok-build-secure-permissions
description: Apply, review, and remove application-level permission guardrails for Grok Build on Windows. Use for safer default tool permissions, focused high-risk shell denies, best-effort sensitive-file restrictions, project-level dontAsk mode, or permission configuration review. This is not a kernel security boundary.
license: MIT
compatibility: Grok Build on native Windows; statically reviewed against public version 0.2.105.
---

# Grok Build Secure Permissions (Windows)

## Boundary

Native Windows currently has no Grok kernel sandbox. Landlock and Seatbelt are
Linux/macOS mechanisms. Treat this skill as an application-layer guardrail that
reduces common accidents, never as unbreakable isolation or secret protection.

For high-sensitivity work, require external containment such as Windows
Sandbox, a disposable VM, a low-privilege account, restricted networking, and a
temporary repository.

Read [references/source-baseline.md](references/source-baseline.md) when
checking source claims, updating this skill for a new Grok Build version, or
reporting the verification level.

## Execution contract

Classify the request before acting:

- `REVIEW`: inspect and report only. Do not modify files.
- `APPLY_DAILY`: merge the balanced global configuration.
- `APPLY_SENSITIVE`: configure project `dontAsk` and disable bypass mode.
- `REMOVE`: restore prior values and remove only entries owned by this skill.

Before any write:

1. Read the existing target files and retain their exact pre-edit contents for
   rollback without displaying unrelated values or secrets.
2. Run `grok inspect --json` when available and inventory every active
   permission source.
3. Preserve unrelated configuration. Never replace an entire TOML or JSON file.
4. Merge into existing tables/objects without duplicate TOML tables, keys, or
   permission rules. Reapplying the skill must be idempotent.
5. If an existing `allow` or `ask` rule conflicts with the requested posture,
   report the exact rule source and obtain direction instead of deleting it.

After a write:

1. Validate TOML/JSON syntax.
2. Run `grok inspect --json` in a new Grok session when possible.
3. If parsing or loading fails, restore the exact pre-edit contents.
4. Report target files, added or changed entries, conflicts, and verification
   level. Never upgrade a load check to runtime verification.

## Design principle

Use **default / dontAsk prompt policy + deny rules**:

- Do not add global `allow` or `ask` rules for `bash` or `edit`.
- Rely on Grok's built-in auto-approval for its fixed read-only command set.
- Let `default` prompt for unmatched operations in daily use.
- Let `dontAsk` deny unmatched operations in sensitive projects.
- Use deny rules as focused filters, not as a complete Windows blocklist.

## Daily global configuration

Merge the following into `%USERPROFILE%\.grok\config.toml`:

```toml
[ui]
default_selected_permission = "allow_once"
remember_tool_approvals = false

[permission]
rules = [
  # Focused high-risk command forms.
  { action = "deny", tool = "bash", pattern = "rm -rf*" },
  { action = "deny", tool = "bash", pattern = "del /s*" },
  { action = "deny", tool = "bash", pattern = "rmdir /s*" },
  { action = "deny", tool = "bash", pattern = "git push*" },
  { action = "deny", tool = "bash", pattern = "git reset --hard*" },
  { action = "deny", tool = "bash", pattern = "format *:" },
  { action = "deny", tool = "bash", pattern = "format.com*" },

  # Best-effort sensitive-path restrictions: direct tools and recognized shell access.
  { action = "deny", tool = "read", pattern = "**/.env" },
  { action = "deny", tool = "read", pattern = "**/.env.*" },
  { action = "deny", tool = "read", pattern = "**/*.pem" },
  { action = "deny", tool = "read", pattern = "**/*.key" },
  { action = "deny", tool = "read", pattern = "**/secrets/**" },
  { action = "deny", tool = "read", pattern = "**/.ssh/**" },
  { action = "deny", tool = "read", pattern = "**/.aws/**" },

  { action = "deny", tool = "edit", pattern = "**/.env" },
  { action = "deny", tool = "edit", pattern = "**/.env.*" },
  { action = "deny", tool = "edit", pattern = "**/*.pem" },
  { action = "deny", tool = "edit", pattern = "**/*.key" },
  { action = "deny", tool = "edit", pattern = "**/secrets/**" },
  { action = "deny", tool = "edit", pattern = "**/.ssh/**" },
  { action = "deny", tool = "edit", pattern = "**/.aws/**" },
]
```

Expected daily behavior, absent other merged rules, hooks, or mode overrides:

- Built-in read-only operations normally auto-approve.
- Other Bash/Edit operations follow the default prompt policy.
- A matching deny rule rejects the operation.

## Deny-rule limitations

- Deny has the highest evaluator priority only after its pattern matches.
- Bash patterns are case-sensitive raw prefix/glob matches even though Windows
  command resolution is commonly case-insensitive.
- Wrappers, aliases, alternate paths, and argument order can change matching.
- Path matching depends on the path representation passed to the evaluator.
- Do not expand this list and claim complete dangerous-command coverage.

## Sensitive project mode

Merge this object into the project's `.claude/settings.local.json` for a
personal policy, or `.claude/settings.json` for an intentionally shared policy:

```json
{
  "permissions": {
    "defaultMode": "dontAsk"
  }
}
```

Also merge this into `%USERPROFILE%\.grok\requirements.toml`:

```toml
[ui]
disable_bypass_permissions_mode = true
```

The user-level requirements file prevents accidental bypass switching but can
still be edited by that user. It is not administrator-enforced policy.

Sensitive-mode expectations are conditional on there being no other merged
`ask` rule or CLI mode override. Built-in auto-approved operations still pass;
unmatched operations are denied without prompting.

## Verification

Run in a new session:

```powershell
grok inspect --json
```

This proves only that configuration sources and rules loaded. Report:

```text
CONFIG_SYNTAX_VALID
Configuration syntax is valid.

CONFIG_LOADED
Grok reports that the configuration loaded.

POLICY_RUNTIME_BEHAVIOR_UNVERIFIED
Concrete tool-call decisions were not dynamically tested.

EXTERNAL_CONTAINMENT_NOT_VERIFIED
VM, account, network, and OS containment were not verified.
```

Only after harmless representative tests in a disposable project may the
result be upgraded to:

```text
POLICY_CANARY_PASSED
Representative permission behavior tests passed.
```

## Break-glass behavior

- Prefer the interactive **Allow once** decision.
- Do not describe `--allow Bash(...)` as an exact command exception. Bash allow
  rules use prefix/glob matching against the whole command string.
- If the user explicitly requests a CLI allow, show its exact scope before use
  and keep it to that run.
- Treat `/always-approve` as an emergency global bypass only.

## Removal

1. Remove only permission rules added by this skill. Remove the
   `[permission]` table only if the skill created it and it becomes empty.
2. Restore prior `[ui]` values when they existed; otherwise remove only
   `default_selected_permission` and `remember_tool_approvals` added here.
3. Restore or remove only the `disable_bypass_permissions_mode` value owned by
   this skill in `%USERPROFILE%\.grok\requirements.toml`.
4. Remove only the `permissions.defaultMode` value owned by this skill from the
   project settings file. Delete that file only if this skill created it and it
   contains nothing else.
5. Start a new session and run `grok inspect --json`. Confirm the skill-owned
   sources and entries are absent; do not require all unrelated rule counts to
   become zero.
