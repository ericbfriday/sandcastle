Status: ready-for-agent

# Support provider-native resumeSession without host session transfer

## Problem Statement

As a Sandcastle user, I can now pass `resumeSession` to agent providers that resume server-side, but the default bind-mount sandbox provider path still assumes a host-captured Claude Code agent session JSONL exists and must be transferred into the sandbox before iteration 1 starts. In practice, this makes `run({ agent: kiro(...), resumeSession })` fail during session resume instead of resuming the agent session, so the advertised resume behavior is inconsistent across agent providers and broken in a common sandbox flow.

## Solution

Make `resumeSession` work consistently across agent providers by separating Claude-style host-captured agent session resume from provider-native server-side resume. Sandcastle should only transfer an agent session between the host and sandbox when the agent provider actually stores a resumable agent session on the host. Providers that resume natively through their own CLI flags should skip host agent session transfer entirely while still receiving the `resumeSession` value during iteration 1.

## User Stories

1. As a Sandcastle user, I want `resumeSession` to work with `kiro()`, so that I can continue a prior agent session without creating a fake Claude Code session file on the host.
2. As a Sandcastle user, I want `run()` to apply the same resume semantics in validation and orchestration, so that a configuration accepted up front does not fail later in iteration setup.
3. As a Sandcastle user, I want bind-mount sandbox provider runs to resume only when the selected agent provider supports that resume path, so that Sandcastle does not attempt the wrong transport mechanism.
4. As a Sandcastle user, I want Claude Code resume to keep using host-captured agent session transfer, so that existing captured agent session workflows continue to work unchanged.
5. As a Sandcastle user, I want provider-native resume to skip host JSONL checks and host-to-sandbox copying, so that server-side agent sessions can resume without local artifacts.
6. As a Sandcastle user, I want resume behavior to apply only to iteration 1, so that Sandcastle preserves its existing single-iteration resume semantics.
7. As a Sandcastle user, I want resume failures to reflect the actual failing mechanism, so that I can tell whether the problem is a missing host session file or a provider-native resume failure.
8. As a Sandcastle user, I want `createWorktree()` and `run()` to agree on when a host agent session file is required, so that the same agent session can be resumed consistently across orchestration entry points.
9. As a maintainer, I want the agent provider contract to describe how resume works, so that adding a new provider does not accidentally inherit Claude-specific agent session behavior.
10. As a maintainer, I want resume-related branching in orchestration to be driven by an explicit provider capability, so that bind-mount sandbox providers do not need to infer resume behavior indirectly.
11. As a maintainer, I want tests for both host-captured and provider-native resume flows, so that regressions are caught when new agent providers are added.
12. As a maintainer, I want non-capturing providers to stay out of host agent session transfer paths, so that `captureSessions: false` providers do not fail on missing Claude-style session storage.
13. As a maintainer, I want agent session capture and agent session resume semantics to remain distinct concepts, so that disabling capture does not automatically imply unsupported resume.
14. As a maintainer, I want iteration results for Claude Code to keep reporting captured agent session metadata where supported, so that existing logging and usage behavior remains stable.
15. As a maintainer, I want the documentation and inline API comments to reflect the actual resume contract, so that users know which agent providers require host-captured sessions and which resume server-side.

## Implementation Decisions

- Introduce an explicit resume capability contract on the agent provider interface that distinguishes host-captured resume from provider-native resume. Avoid treating `captureSessions` as the orchestration decision for whether pre-iteration agent session transfer should occur.
- Keep Claude Code on the host-captured resume path. Claude Code remains the provider whose agent session can be resumed from host-side JSONL content and whose captured agent session can be pulled back from the sandbox after an iteration.
- Keep Kiro on the provider-native resume path. Kiro should receive the `resumeSession` identifier in its CLI arguments for iteration 1, but Sandcastle should not attempt host-to-sandbox agent session transfer before invocation.
- Centralize the resume decision used by validation and orchestration so both entry points enforce the same behavior. Sandcastle should not accept a `resumeSession` configuration that the iteration layer cannot honor, and it should not reject a provider-native resume path that needs no host artifact.
- Preserve the existing rule that `resumeSession` applies only to iteration 1 and remains incompatible with `maxIterations > 1`.
- Preserve the existing distinction between bind-mount sandbox provider behavior and isolated sandbox provider behavior, but ensure bind-mount-specific agent session transfer only runs when the provider requires host-captured resume.
- Keep agent session capture as a separate concern from resume capability. A provider may support provider-native resume without host capture, and future providers may support neither, one, or both capabilities.
- Ensure resume-related status output and errors describe the correct path being taken. If Sandcastle is performing host agent session transfer, failures should mention that transfer path; if Sandcastle is delegating resume to the provider, failures should come from the agent invocation path instead.
- Update API comments and developer-facing documentation for `resumeSession` so they explain the difference between host-captured agent session resume and provider-native resume.

## Testing Decisions

- Good tests should assert externally observable behavior of the orchestration APIs rather than internal implementation details. In practice, that means checking whether a run is accepted or rejected, whether the correct resume mechanism is invoked, whether host agent session files are required, and whether existing Claude Code capture behavior still works.
- Test `run()` validation for both resume families: host-captured providers should still require a host agent session file, while provider-native resume should not.
- Test `createWorktree()` validation for the same matrix, so both orchestration entry points share the same `resumeSession` contract.
- Test `orchestrate()` for bind-mount sandbox provider resume with a host-captured provider, verifying that iteration 1 transfers the agent session into the sandbox and that post-iteration capture behavior remains intact.
- Test `orchestrate()` for bind-mount sandbox provider resume with a provider-native provider, verifying that no host-to-sandbox agent session transfer occurs and the provider still receives the `resumeSession` value during iteration 1.
- Test that non-capturing providers remain outside the host agent session transfer path even when `resumeSession` is set, covering the regression reported in the review.
- Test that Claude Code-style capture and usage extraction still behave as before after the resume contract is separated.
- Use the existing orchestration and provider tests around `resumeSession`, `captureSessions`, and captured agent session metadata as prior art, extending them to cover the provider-native bind-mount resume case rather than introducing a completely different testing style.

## Out of Scope

- Changing the rule that `resumeSession` cannot be combined with `maxIterations > 1`.
- Adding resume support to agent providers that currently ignore `resumeSession`.
- Redesigning isolated sandbox provider resume semantics beyond keeping the capability model coherent.
- Changing how agent session usage is parsed or reported except where required to preserve current Claude Code behavior.
- Broader refactors to unrelated prompt, branching, or sandbox lifecycle code.

## Further Notes

- The key regression is that Sandcastle now advertises provider-native resume for Kiro during validation, but the bind-mount sandbox provider orchestration path still behaves as if every resumable agent session is a Claude Code JSONL that must be copied into the sandbox.
- This work should prefer a deep module boundary around resume capability and orchestration policy so future agent providers do not need to duplicate Claude-specific assumptions.
- The project glossary should continue to use Sandcastle terminology such as agent, sandbox, bind-mount sandbox provider, iteration, and agent session throughout the implementation and follow-up issue discussion.
