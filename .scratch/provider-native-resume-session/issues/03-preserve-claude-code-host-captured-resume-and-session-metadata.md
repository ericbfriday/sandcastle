Status: ready-for-agent

# Preserve Claude Code host-captured resume and session metadata

## Parent

`.scratch/provider-native-resume-session/PRD.md`

## What to build

Protect the existing Claude Code `resumeSession` path after the resume capability split. Sandcastle should continue to transfer a host-captured agent session into the sandbox for iteration 1 when the provider requires it, capture the updated agent session back to the host afterward, and keep reporting captured agent session metadata such as the host session file path and usage when available.

This slice should also update the developer-facing `resumeSession` contract so the difference between provider-native resume and host-captured resume is explicit in API comments and related documentation.

## Acceptance criteria

- [ ] Claude Code `resumeSession` still uses host-to-sandbox agent session transfer for iteration 1 and still captures the resulting agent session back to the host.
- [ ] Captured agent session metadata, including host session file paths and usage extraction where supported, remains intact for the Claude Code path.
- [ ] Tests and documentation clearly cover the two resume families so future agent providers do not inherit Claude-specific resume assumptions by accident.

## Blocked by

- `.scratch/provider-native-resume-session/issues/01-support-provider-native-resume-session-in-run-bind-mount-flows.md`
