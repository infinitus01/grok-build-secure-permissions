# Grok Build Secure Permissions (Windows)

Unofficial application-level permission guardrails for [Grok Build](https://x.ai/cli) on Windows.

> **Important**
>
> This community project is not affiliated with or endorsed by xAI. It is not a
> kernel sandbox. Grok's Landlock and Seatbelt enforcement applies to Linux and
> macOS, not native Windows. These rules reduce common accidental high-risk
> actions, but they do not provide unbreakable data isolation, secret
> protection, or an OS security boundary.

For high-sensitivity work, add external isolation such as Windows Sandbox, a
disposable VM, a low-privilege account, restricted networking, and temporary
test repositories.

## What it does

- Applies a conservative daily policy based on Grok's default prompt mode and
  focused deny rules.
- Adds highest-priority deny rules for selected high-risk command forms.
- Adds best-effort Read/Edit restrictions for common sensitive paths.
- Separates daily operation from project-level `dontAsk` mode.
- Defines safe review, apply, verification, rollback, and removal procedures.

## Design principle

Use **default / dontAsk prompt policy + deny rules**:

- Do not add global `allow` or `ask` rules for `bash` or `edit`.
- Let Grok's built-in auto-approval handle its fixed read-only command set.
- Let the active permission mode decide what happens to unmatched operations.
- Use explicit `deny` rules only as focused application-level filters.

## Installation

### User-wide

Create this directory:

```text
%USERPROFILE%\.grok\skills\grok-build-secure-permissions\
```

Copy `SKILL.md` and the `references` directory into it.

### Project-only

Copy those files to:

```text
<repository>\.grok\skills\grok-build-secure-permissions\
```

Start a new Grok session and invoke:

```text
/grok-build-secure-permissions
```

Then inspect skill discovery and configuration sources:

```powershell
grok inspect --json
```

## Modes

| Mode | How to enable | Expected behavior |
|---|---|---|
| Daily | Global configuration only | Built-in safe commands normally auto-pass; unmatched operations follow the default prompt policy. |
| Sensitive | Project `defaultMode: "dontAsk"` plus bypass disabled | Unmatched operations are denied, provided no other merged `ask` rule or CLI mode override applies. |

A user-level `requirements.toml` prevents accidental bypass switching but
remains editable by that user. It is not administrator-enforced containment.

## Limitations

- Bash deny rules use case-sensitive raw prefix/glob matching.
- Path restrictions depend on the path representation received by the
  permission evaluator and remain best-effort.
- The deny list is deliberately non-exhaustive.
- Native Windows has no Grok kernel sandbox.
- `grok inspect` proves configuration loading, not the runtime decision for a
  specific tool call.
- Custom `Bash(...)` allow rules are not exact-command grants; they match the
  whole command string by prefix/glob semantics.

## Verification levels

The skill distinguishes:

- `CONFIG_SYNTAX_VALID` — configuration syntax is valid.
- `CONFIG_LOADED` — Grok reports that the configuration loaded.
- `POLICY_RUNTIME_BEHAVIOR_UNVERIFIED` — concrete tool decisions were not dynamically tested.
- `EXTERNAL_CONTAINMENT_NOT_VERIFIED` — VM, account, network, or OS isolation was not verified.
- `POLICY_CANARY_PASSED` — representative harmless tests passed in a disposable project.

## Audit baseline

This revision was statically reviewed against:

- Public repository: `xai-org/grok-build`
- Public commit: `7cfcb20d2b50b0d18801a6c0af2e401c0e060894`
- Upstream `SOURCE_REV`: `f9736c7b86f8e1c0e99e20ebbbd1195cd0c147e3`
- Declared package version: `0.2.105`
- Dynamic reproduction: not performed

See [references/source-baseline.md](references/source-baseline.md) for the
fixed evidence links and verification boundary. Future Grok Build versions may
change these semantics and should be re-audited before this configuration is
treated as current.

## License

MIT License. See [LICENSE](LICENSE).

## Disclaimer

This project is based on static analysis of public source code and official
documentation. It has not undergone large-scale dynamic security or penetration
testing. Use it as a focused guardrail, not as a security certification.
