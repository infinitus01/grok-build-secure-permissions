# Source Baseline

## Fixed identity

```text
PUBLIC_REPOSITORY: xai-org/grok-build
PUBLIC_COMMIT: 7cfcb20d2b50b0d18801a6c0af2e401c0e060894
SOURCE_REV: f9736c7b86f8e1c0e99e20ebbbd1195cd0c147e3
PACKAGE_VERSION: 0.2.105
REVIEW_TYPE: PUBLIC_STATIC_SOURCE_AND_DOCUMENTATION_REVIEW
DYNAMIC_REPRODUCTION: NOT_CONFIRMED
RUNTIME_EFFECT: NOT_CONFIRMED
```

This file records the evidence boundary for the accompanying skill. It does not
claim that source version `0.2.105` was distributed through every official
binary channel.

## Source-confirmed behavior

### Permission evaluation

- Permission rules precede remembered grants, built-in auto-approvals, and the
  prompt policy.
- Severity is `deny` over `ask` over `allow`.
- `dontAsk` denies unmatched operations but does not disable built-in
  auto-approvals.
- A user-level requirements file can disable bypass mode, but remains under the
  user's control.

Evidence:

- [Permissions and Safety Controls](https://github.com/xai-org/grok-build/blob/7cfcb20d2b50b0d18801a6c0af2e401c0e060894/crates/codegen/xai-grok-pager/docs/user-guide/22-permissions-and-safety.md)
- [Permission manager](https://github.com/xai-org/grok-build/blob/7cfcb20d2b50b0d18801a6c0af2e401c0e060894/crates/codegen/xai-grok-workspace/src/permission/manager.rs)

### Bash matching

- Custom Bash allow rules match the whole raw command string by prefix/glob.
- Deny and ask checks also inspect split command segments.
- Built-in read-only commands use a separate per-segment, word-boundary list.

Evidence:

- [Permissions rule matching reference](https://github.com/xai-org/grok-build/blob/7cfcb20d2b50b0d18801a6c0af2e401c0e060894/crates/codegen/xai-grok-pager/docs/user-guide/22-permissions-and-safety.md#rule-matching-reference)
- [Permission policy evaluator](https://github.com/xai-org/grok-build/blob/7cfcb20d2b50b0d18801a6c0af2e401c0e060894/crates/codegen/xai-grok-workspace/src/permission/policy.rs)

### Sensitive paths

- Direct Read/Edit checks match the supplied path string without full
  canonicalization or symlink resolution.
- Read/Edit deny rules also apply to recognized paths touched by shell commands;
  that shell-level check resolves symlinks.

Evidence:

- [Read, Edit, and Grep rule semantics](https://github.com/xai-org/grok-build/blob/7cfcb20d2b50b0d18801a6c0af2e401c0e060894/crates/codegen/xai-grok-pager/docs/user-guide/22-permissions-and-safety.md#read-edit-and-grep-rules)
- [Shell file-access evaluator](https://github.com/xai-org/grok-build/blob/7cfcb20d2b50b0d18801a6c0af2e401c0e060894/crates/codegen/xai-grok-workspace/src/permission/shell_access.rs)

### Windows sandbox boundary

- Documented kernel enforcement is Linux Landlock and macOS Seatbelt.
- Non-Unix enforcement can report unavailable without establishing a kernel
  containment boundary.

Evidence:

- [Sandbox documentation](https://github.com/xai-org/grok-build/blob/7cfcb20d2b50b0d18801a6c0af2e401c0e060894/crates/codegen/xai-grok-pager/docs/user-guide/18-sandbox.md)
- [Sandbox platform implementation](https://github.com/xai-org/grok-build/blob/7cfcb20d2b50b0d18801a6c0af2e401c0e060894/crates/codegen/xai-grok-sandbox/src/lib.rs)

### Skill discovery

- User skills: `~/.grok/skills/<name>/SKILL.md`.
- Project skills: `<repo_root>/.grok/skills/<name>/SKILL.md`.
- `grok inspect --json` reports discovered skill sources.

Evidence:

- [Grok Skills documentation](https://github.com/xai-org/grok-build/blob/7cfcb20d2b50b0d18801a6c0af2e401c0e060894/crates/codegen/xai-grok-pager/docs/user-guide/08-skills.md)

## Revalidation rule

For a later Grok Build revision:

1. Pin the new full public commit and its `SOURCE_REV`.
2. Compare every evidence file above across the commit range.
3. Mark each existing claim `FIXED`, `PARTIAL`, `UNCHANGED`, `REGRESSED`, or
   `UNVERIFIED`; add `NEW` findings separately.
4. Do not infer behavior from renamed files, comments, or release notes alone.
5. Keep static reachability, dynamic reproduction, and runtime effect separate.
