Status: ready-for-agent

# Support provider-native `resumeSession` in `run()` bind-mount flows

## Parent

`.scratch/provider-native-resume-session/PRD.md`

## What to build

Make provider-native `resumeSession` work end-to-end through the default `run()` bind-mount sandbox provider flow. Sandcastle should pass the `resumeSession` identifier into the agent provider for iteration 1 without trying to transfer a host-captured agent session into the sandbox when the provider resumes server-side.

This slice should introduce the explicit resume capability needed to distinguish provider-native resume from host-captured resume, then use that capability to make the `run()` and `orchestrate()` bind-mount path verifiably correct for `kiro()` and other providers with the same resume shape.

## Acceptance criteria

- [ ] `run({ agent: kiro(...), resumeSession })` works through the bind-mount sandbox provider path without requiring a host Claude Code session file.
- [ ] The iteration 1 bind-mount orchestration path skips host-to-sandbox agent session transfer for provider-native resume while still passing the `resumeSession` identifier to the agent provider.
- [ ] Tests cover the end-to-end regression: provider-native resume is accepted in `run()` and does not enter the host agent session transfer path.

## Blocked by

None - can start immediately
