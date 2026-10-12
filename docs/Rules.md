
# Comrad — Development Rules

These rules are mandatory for all contributors and AI coding agents working on Comrad.

---

## 1. Project Identity

Comrad is a native macOS companion for monitoring AI coding agents running in CLI environments.

Primary technologies:
- Swift
- SwiftUI
- AppKit
- Foundation
- Observation
- UserNotifications
- XCTest

Primary integrations:
- Claude Code
- OpenCode
- Agy

The application must prioritize reliable monitoring, minimal distraction, local-first processing, and user control.

## 2. Source of Truth

Before implementing a feature, read:

1. `PRD.md`
2. `TRD.md`
3. `design.md`
4. `rules.md`

The PRD defines product behavior.

The TRD defines architecture and technical contracts.

The design document defines visual and interaction behavior.

This file defines implementation constraints and engineering rules.

If documents conflict, stop and identify the conflict rather than silently inventing a new requirement.

## 3. General Engineering Principles

- Prefer simple, modular implementations.
- Keep changes limited to the requested scope.
- Do not introduce unnecessary dependencies.
- Do not duplicate existing functionality.
- Do not replace working architecture without a documented reason.
- Do not create abstractions without a practical use.
- Prefer explicit state transitions over scattered Boolean flags.
- Keep side effects outside pure domain logic.
- Make errors observable and actionable.
- Never claim that a feature works without testing it.

## 4. Swift Coding Standards

- Use clear, descriptive type and function names.
- Follow standard Swift naming conventions.
- Prefer `struct` for value models.
- Use `enum` for finite state sets.
- Use protocols for genuine integration boundaries.
- Use `final class` when subclassing is not intended.
- Use dependency injection for external services.
- Avoid force unwrapping unless the invariant is explicit and guaranteed.
- Avoid `try!` and `as!` in production code.
- Handle errors explicitly.
- Avoid excessively large files and types.
- Avoid deeply nested conditionals.
- Prefer small functions with one clear responsibility.
- Document non-obvious decisions and invariants.

## 5. Concurrency Rules

- Respect Swift concurrency and actor isolation.
- Keep UI-facing state on the main actor.
- Do not block the main thread with file or database operations.
- Use actors or equivalent synchronization for shared mutable state where appropriate.
- Avoid unnecessary detached tasks.
- Do not assume an asynchronous event arrives in chronological order.
- Make event processing safe under concurrent delivery.
- Cancel owned tasks during shutdown when appropriate.
- Do not introduce unstructured background tasks without a lifecycle plan.

## 6. Architecture Rules

Maintain these boundaries:

### UI
Responsible for:
- Rendering views.
- Handling user interaction.
- Showing loading, error, and empty states.
- Animating presentation changes.

The UI must not parse agent-specific JSON.

### Domain
Responsible for:
- Agent and session models.
- Event types.
- State transitions.
- Permission request lifecycle.
- Domain invariants.

Domain logic should remain testable without launching the app.

### Integration
Responsible for:
- Agent-specific APIs.
- Hook and plugin integration.
- Native payload decoding.
- Mapping native events into normalized events.
- Reporting supported capabilities.

### Infrastructure
Responsible for:
- Local IPC.
- Persistence.
- Notifications.
- Logging.
- Configuration.

Do not let UI components access integration internals directly.

## 7. Agent Integration Rules

- Verify the official API before implementing an adapter.
- Check the installed and supported agent versions.
- Use documented hooks, plugin APIs, or SDKs where available.
- Never assume all agents emit the same event schema.
- Keep agent-specific code inside its adapter.
- Use normalized domain events after the adapter boundary.
- Treat unknown event types as unsupported, not successful.
- Validate all identifiers and required fields.
- Handle missing session information explicitly.
- Do not rely on terminal screenshots or OCR as the primary integration.
- Label heuristic detection as uncertain.
- Do not claim unsupported capabilities.
- Keep adapter tests independent of live agent sessions.

## 8. Permission and Safety Rules

These rules are non-negotiable.

1. Never automatically approve a command or permission.
2. Never bypass an agent's permission system.
3. Never execute shell commands received from event payloads.
4. Never treat a notification dismissal as permission approval.
5. Never treat card dismissal as request resolution.
6. Never clear a pending request without a valid matching resolution signal or explicit supported user action.
7. Never fabricate an approval, completion, or success event.
8. Never report an inferred state as a confirmed state.
9. Do not expose sensitive command arguments unnecessarily.
10. Do not introduce remote command execution.

Comrad is an observer and supervisor.

Any future approval workflow must use an explicitly supported API, identify the exact pending request, verify the current state, and require a deliberate user action.

## 9. Event Processing Rules

- Validate input before processing.
- Enforce payload size limits.
- Use stable event IDs where available.
- Deduplicate repeated events.
- Preserve request correlation IDs.
- Handle out-of-order events.
- Keep session state independent.
- Persist important state transitions.
- Reject malformed events safely.
- Avoid logging full payloads containing sensitive data.
- Treat inactivity as inactivity, not proof of completion.
- Treat a missing heartbeat as stale or disconnected, not automatically failed.
- Do not let an older progress event overwrite a newer confirmed state.

## 10. UI and Design Rules

- Follow `design.md`.
- Keep the compact pill unobtrusive.
- Use the expanded card for actionable details.
- Use consistent spacing, typography, and corner radii.
- Respect system appearance and accessibility settings.
- Support reduced motion.
- Do not communicate status using color alone.
- Keep the interface responsive to changing content.
- Avoid unnecessary animations and excessive visual effects.
- Preserve pending requests when the card closes.
- Do not block the entire app while one agent is waiting.
- Provide a fallback interface when notch positioning is unavailable.

## 11. AppKit and Window Rules

- Keep panel creation and lifecycle management inside the panel controller.
- Keep screen-positioning logic inside the positioner.
- Do not hardcode a specific MacBook resolution.
- Recalculate position after display changes.
- Test full-screen and multi-display behavior.
- Do not assume floating window levels guarantee visibility in every context.
- Keep a reliable menu bar or standard-window fallback.
- Ensure the app can be accessed and quit even if the floating panel fails.

## 12. Persistence Rules

- Keep persistence behind a dedicated interface.
- Use schema migrations for persistent format changes.
- Bound the amount of retained history.
- Avoid storing complete transcripts by default.
- Do not persist credentials in ordinary event logs.
- Preserve unresolved requests across application restarts.
- Never silently resolve pending requests during restoration.
- Handle corrupt or incompatible records safely.

## 13. Privacy and Security Rules

- Local-first behavior is the default.
- No telemetry without an explicit product decision and user disclosure.
- Restrict access to local communication channels.
- Validate event origin where practical.
- Use least-privilege permissions.
- Avoid unnecessary Accessibility or screen-recording permissions.
- Never log authentication tokens or secrets.
- Redact sensitive values from diagnostics.
- Never overwrite unrelated agent configuration.
- Only remove integration files owned by Comrad.
- Ask for explicit user consent before installing integrations.
- Document all external data flows.

## 14. Testing Rules

Every non-trivial change must include relevant tests.

Minimum expectations:
- New event types require decoder and reducer tests.
- New state transitions require positive and negative tests.
- New adapter mappings require representative fixtures.
- Persistence changes require migration and restoration tests.
- Notification changes require request-identity tests.
- UI state changes require transition tests where practical.

Test failure conditions, not only successful execution.

Do not remove or weaken a test simply to make a build pass.

Do not claim tests passed unless they were actually executed.

## 15. Git and Repository Rules

- Keep changes focused.
- Use descriptive commit messages.
- Do not commit secrets, credentials, or local event logs.
- Do not commit build products or generated user data.
- Do not overwrite unrelated changes.
- Inspect the diff before committing.
- Preserve existing project configuration unless a change is required.
- Update documentation when behavior or architecture changes.
- Keep `.gitignore` appropriate for macOS and Xcode.

## 16. AI Coding Agent Workflow

Before changing code:

1. Read the relevant documentation.
2. Inspect the existing implementation.
3. Identify affected modules.
4. State the intended change and assumptions.
5. Identify relevant tests.

During implementation:

1. Make the smallest coherent change.
2. Preserve established architecture.
3. Avoid unrelated refactoring.
4. Add or update tests.
5. Handle errors explicitly.
6. Respect permission and privacy rules.

After implementation:

1. Run the relevant tests.
2. Run the build when available.
3. Inspect compiler warnings.
4. Review the diff.
5. Update documentation.
6. Report files changed.
7. Report tests actually executed.
8. Disclose known limitations.

If an important requirement is ambiguous, inspect the codebase and documentation before guessing.

If the task requires destructive changes, privileged actions, or a change to permission handling, stop and obtain explicit authorization.

## 17. Dependency Rules

- Prefer Apple's frameworks.
- Introduce a dependency only for a clear benefit.
- Check maintenance status and compatibility.
- Review permissions and security implications.
- Avoid dependencies that require unnecessary network access.
- Document important dependencies and their purpose.

## 18. Definition of Done

A change is complete only when:

- It satisfies the intended requirement.
- It follows the architecture.
- It handles failure conditions.
- It has appropriate tests.
- It does not violate safety or privacy rules.
- Its documentation is current.
- Its limitations are accurately reported.

## 19. Final Rule

Reliability and user control take priority over convenience.

Comrad must never hide uncertainty, fabricate agent state, or silently make a permission decision on behalf of the user.
