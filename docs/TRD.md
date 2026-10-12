
# Comrad — Technical Requirements Document

**Product:** Comrad  
**Platform:** macOS  
**Language:** Swift  
**UI:** SwiftUI + AppKit  
**Architecture:** Modular, event-driven, local-first  
**Version:** 1.0

---

## 1. Technical Objectives

The system must:

1. Receive events from supported AI coding agents.
2. Normalize agent-specific payloads into a shared schema.
3. Maintain accurate state for multiple concurrent sessions.
4. Display changes in a native macOS floating interface.
5. Notify users about actionable events.
6. Persist outstanding requests and bounded activity history.
7. Isolate agent-specific implementation details.
8. Recover gracefully from malformed events and integration failures.

## 2. Technology Stack

### Required

- Swift
- SwiftUI
- AppKit
- Foundation
- Observation framework
- UserNotifications
- XCTest
- Xcode

### Optional, introduced when justified

- SQLite for persistent event history.
- SwiftData for simpler application-owned persistence.
- A TypeScript/Node.js hook receiver where an agent's integration API requires it.

Do not introduce a package dependency unless it solves a concrete requirement.

## 3. Architecture

Use a layered, event-driven architecture.

    Agent CLI
       |
       v
    Agent-specific Adapter
       |
       v
    Local Event Bridge
       |
       v
    Event Decoder and Validator
       |
       v
    Event Normalizer
       |
       v
    Session State Reducer
       |
       +----> Persistence
       |
       +----> Notification Service
       |
       v
    Observable App State
       |
       +----> Floating NSPanel
       +----> Activity Card
       +----> History and Settings

### Architectural rules

- Adapters must not depend on SwiftUI views.
- Views must not parse agent-specific payloads.
- The event reducer must be deterministic and independently testable.
- External input must be validated before state changes.
- UI state and persisted domain state must remain conceptually separate.
- Permission requests must have explicit identities and lifecycle states.
- Integration failures must be observable.

## 4. Application Lifecycle

### App entry point

`ComradApp.swift` initializes the SwiftUI application and shared dependencies.

### App delegate

`AppDelegate.swift` manages application lifecycle integration where needed.

Responsibilities include:
- Creating the panel controller.
- Starting the event bridge.
- Initializing persistence.
- Restoring unresolved requests.
- Registering notification actions.
- Handling shutdown and cleanup.

Use a menu bar entry point for recovery, settings, and quitting.

If `LSUIElement` is enabled, ensure the user can still access these controls.

## 5. Floating Window Architecture

### Components

- `NotchPanelController`
- `NotchPositioner`
- `NotchContentView`
- `AppState`

### Panel implementation

Use `NSPanel` with an `NSHostingView` hosting the SwiftUI content.

The panel must support:

- Borderless appearance.
- Transparent background.
- Non-activating presentation where appropriate.
- Floating window level.
- Configurable dimensions.
- Hover detection.
- Click-to-expand behavior.
- Dismissal without resolving pending requests.
- Screen-aware positioning.
- Multiple-display handling.
- A fallback location when notch geometry is unavailable.

### Window positioning

Use `NSScreen` and available display geometry to calculate the panel's location.

Do not hardcode screen resolution.

The positioner must:
1. Select the intended screen.
2. Calculate the compact panel frame.
3. Anchor expansion consistently.
4. Recalculate after screen configuration changes.
5. Keep the panel inside usable display bounds.

Window-level and full-screen behavior must be tested on real hardware.

## 6. Domain Models

### AgentKind

Supported values:

- `claudeCode`
- `openCode`
- `agy`
- `unknown`

### AgentEventType

Initial values:

- `sessionStarted`
- `progress`
- `toolStarted`
- `toolCompleted`
- `permissionRequired`
- `permissionResolved`
- `inputRequired`
- `inputResolved`
- `sessionIdle`
- `sessionCompleted`
- `sessionFailed`
- `integrationDisconnected`
- `heartbeat`

Adapters may map native events into these values only when their semantics match.

### AgentEvent

The normalized event must contain:

- Stable event ID.
- Schema version.
- Agent kind.
- Agent session ID.
- Project identifier or path when available.
- Event timestamp.
- Event type.
- Short summary.
- Optional structured details.
- Optional request correlation ID.
- Optional source metadata.

Do not include arbitrary environment variables, credentials, or full transcripts.

### AgentSession

The session model must contain:

- Comrad session identity.
- Agent kind.
- Agent-provided session identity.
- Project path or identifier.
- Current status.
- Last observed activity.
- Last confirmed event time.
- Latest summary.
- Outstanding request IDs.
- Integration health.
- Optional launch context.

### PermissionRequest

The request model must contain:

- Stable request identity.
- Associated session identity.
- Request type.
- Human-readable description.
- Creation time.
- Current status.
- Optional expiry or stale indicator.
- Resolution timestamp when available.
- Resolution source.

A notification dismissal is not a request resolution.

## 7. Event Processing Pipeline

The event-processing pipeline must:

1. Receive an event.
2. Enforce payload size limits.
3. Decode the payload.
4. Validate the schema and required fields.
5. Validate the event source.
6. Normalize the agent-specific representation.
7. Deduplicate using stable identifiers where available.
8. Resolve the correct session.
9. Apply the event through the state reducer.
10. Persist important state changes.
11. Publish UI updates.
12. Schedule an appropriate notification if required.

Invalid events must be rejected and logged without exposing sensitive payloads.

## 8. Session State Machine

Initial states:

- `discovered`
- `running`
- `waitingForInput`
- `waitingForPermission`
- `idle`
- `completed`
- `failed`
- `disconnected`

### Transition rules

- Session start -> running.
- Progress or tool activity -> running, unless a stronger unresolved state exists.
- Explicit input request -> waiting for input.
- Explicit permission request -> waiting for permission.
- Matching request resolution -> recalculate state from current session information.
- Idle event -> idle unless another unresolved request takes precedence.
- Confirmed completion -> completed.
- Confirmed failure -> failed.
- Lost integration connectivity -> disconnected or stale.

Do not infer completion solely from inactivity, a missing heartbeat, a closed UI card, or an idle event.

The reducer must handle out-of-order and duplicate events.

## 9. Agent Adapter Interface

Define a common adapter contract.

Each adapter must support, where applicable:

- Installation and configuration validation.
- Removal of Comrad-managed integration components.
- Agent capability discovery.
- Native event ingestion.
- Event normalization.
- Integration health reporting.
- Version compatibility checks.

Adapters must declare unsupported capabilities.

### Claude Code adapter

Use documented hooks and supported lifecycle events. Validate the actual input schema against the installed Claude Code version.

### OpenCode adapter

Use the documented plugin API and verify event names and payload structures against the supported OpenCode version.

### Agy adapter

Implement only after confirming the actual CLI's supported extension points. Use an official hook, SDK, plugin, or supported wrapper where available.

## 10. Local Event Bridge

### Initial implementation

A JSONL-based event stream may be used for the first prototype.

Requirements:
- Complete-line parsing.
- Bounded payload sizes.
- Safe handling of partial writes.
- Duplicate detection.
- File rotation.
- Recovery after app restart.
- Restrictive filesystem permissions.
- No execution of commands embedded in events.

Multiple writers must not corrupt the event stream. Use separate per-producer streams or an explicit serialization strategy.

### Production implementation

Prefer a Unix domain socket for low-latency event delivery when the agent integrations support it.

The bridge must:
- Run locally.
- Authenticate or restrict local clients where practical.
- Validate every message.
- Handle reconnects.
- Enforce message size limits.
- Avoid blocking agent hook processes unnecessarily.
- Report its health to the application.

Do not expose an unauthenticated event receiver to the network.

## 11. Persistence

Persist:

- Agent registrations and settings.
- Session metadata.
- Outstanding attention requests.
- Bounded event history.
- Integration health and last-seen timestamps.

Use SQLite or SwiftData according to actual query, migration, and concurrency requirements.

### Persistence rules

- Keep persistence behind a repository interface.
- Do not perform synchronous database work on the main thread.
- Use schema migrations.
- Apply a configurable retention policy.
- Store the minimum necessary event details.
- Restore unresolved requests safely after a restart.
- Never automatically resolve requests simply because the application restarted.

## 12. Notification Service

Use `UNUserNotificationCenter`.

Support:
- Permission-required notifications.
- Input-required notifications.
- Failure notifications.
- Optional completion notifications.
- User-controlled notification preferences.

Notifications must use stable identifiers where possible.

A matching resolution event should update the relevant pending item. If the event cannot be correlated safely, preserve the request for review rather than clearing an unrelated request.

Handle notification authorization denial gracefully.

## 13. UI State Management

Use a shared observable application model.

The model must expose:
- Registered agents.
- Sessions.
- Pending attention requests.
- Recent activity.
- Integration health.
- Presentation state.
- User preferences.

The view hierarchy must not own the source of truth for session state.

Keep domain-state changes separate from animations and window resizing.

## 14. Security and Privacy

- Keep processing local by default.
- Validate all event input.
- Restrict IPC access.
- Redact sensitive data.
- Avoid storing full transcripts.
- Do not execute commands received from event payloads.
- Do not silently approve permissions.
- Do not modify unrelated agent configuration.
- Require explicit user action for integration installation.
- Remove only Comrad-owned hooks during uninstall.
- Avoid unnecessary Accessibility permissions.
- Use secure storage if future features require credentials.
- Document any future network communication.

## 15. Error Handling

Handle:

- Invalid event schema.
- Missing session identity.
- Duplicate event.
- Unknown agent type.
- Unsupported event type.
- Event bridge failure.
- Permission notification failure.
- Database failure.
- Agent process exit.
- Stale session.
- Configuration conflict.
- Integration version mismatch.

Failures must be observable without causing the entire application to crash.

## 16. Testing

### Unit tests

- Event decoding.
- Payload validation.
- Event normalization.
- Session transitions.
- Request lifecycle.
- Deduplication.
- Out-of-order events.
- Persistence recovery.
- Notification identifier mapping.

### Integration tests

- Claude Code hook to UI.
- OpenCode plugin to UI.
- Agy adapter to UI.
- Bridge reconnect.
- App restart with pending requests.
- Concurrent sessions.

### Manual UI tests

- Compact presentation.
- Hover expansion.
- Click expansion.
- Outside-click dismissal.
- Escape-key dismissal.
- Multiple displays.
- Full-screen applications.
- Sleep and wake.
- Notification authorization denied.

## 17. Performance Targets

Initial targets:

- Local event-to-UI latency under 250 ms under normal conditions.
- Bounded memory use for event history.
- No unnecessary active polling.
- No blocking event processing on the main thread.
- Smooth panel resizing.
- Graceful handling of bursts of events.

Measure actual performance before making release claims.

## 18. Build and Release

Development:
- Xcode Debug configuration.
- XCTest.
- Local sample-event fixtures.

Release:
- Optimized Release configuration.
- Code signing.
- Notarization where required.
- Versioned agent compatibility.
- Clean installation and uninstall tests.
- User documentation.

## 19. Definition of Done

A feature is complete when:

1. Its requirements are documented.
2. Its implementation is isolated and testable.
3. Inputs are validated.
4. Error handling exists.
5. Relevant tests pass.
6. Accessibility is considered.
7. Security implications are reviewed.
8. Unsupported behavior is documented.
9. No permission-safety rules are violated.
