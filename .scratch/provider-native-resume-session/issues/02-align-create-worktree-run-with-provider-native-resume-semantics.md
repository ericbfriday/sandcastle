Status: ready-for-agent

# Align `createWorktree().run()` with provider-native resume semantics

## Parent

`.scratch/provider-native-resume-session/PRD.md`

## What to build

Extend the provider-native `resumeSession` behavior from `run()` to the worktree entrypoint so Sandcastle behaves consistently regardless of whether the caller starts from a worktree or a repo run. A provider-native resume path should not require a host-captured agent session file, should still apply only to iteration 1, and should use the same bind-mount sandbox provider orchestration behavior as the main run path.

This slice should reuse the resume capability introduced for the `run()` path and prove that `createWorktree().run()` now shares the same externally visible `resumeSession` contract.

## Acceptance criteria

- [ ] `createWorktree().run({ agent: kiro(...), resumeSession })` does not require a host Claude Code session file when the agent provider resumes server-side.
- [ ] The worktree entrypoint preserves the existing `resumeSession` iteration 1 semantics and `maxIterations > 1` rejection while using the provider-native resume path where appropriate.
- [ ] Tests show that the worktree entrypoint and `run()` now agree on provider-native versus host-captured resume behavior.

## Blocked by

- `.scratch/provider-native-resume-session/issues/01-support-provider-native-resume-session-in-run-bind-mount-flows.md`
