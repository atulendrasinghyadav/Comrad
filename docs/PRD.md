
# Comrad — Product Requirements Document

**Product:** Comrad  
**Platform:** macOS  
**Product Type:** AI Coding Agent Companion  
**Status:** Initial Product Specification  
**Version:** 1.0

---

## 1. Product Overview

Comrad is a native macOS companion application that monitors AI coding agents running in the command-line interface (CLI).

It helps developers supervise agents without repeatedly switching between terminals, IDEs, and other applications.

Comrad displays a compact pill near the MacBook notch or at an appropriate top-of-screen position. The pill expands on hover and opens a larger activity card when clicked. It provides agent progress updates, alerts when an agent requires user attention, and allows the developer to identify and return to the relevant terminal session.

### Supported agents

- Claude Code
- OpenCode
- Agy

Each agent integration must use its supported hooks, plugin APIs, lifecycle events, or documented integration mechanisms. Unsupported capabilities must be clearly identified rather than simulated.

## 2. Problem Statement

Developers frequently run multiple AI coding agents concurrently.

These agents may:
- Execute commands.
- Read and modify project files.
- Run tests and builds.
- Ask for permission.
- Request additional user input.
- Encounter errors.
- Complete tasks while the developer is doing something else.

Developers can miss permission prompts, lose track of agent progress, and waste time repeatedly checking terminal windows.

Comrad addresses this problem through a persistent, lightweight, centralized activity interface.

## 3. Product Vision

Make AI coding agents easier to supervise by providing a reliable, unobtrusive, and actionable overview of their activity.

Comrad should feel like a natural extension of the macOS desktop rather than another dashboard that requires constant attention.

## 4. Product Goals

1. Display the status of supported AI coding agents in real time.
2. Notify the user when an agent needs permission or explicit input.
3. Show meaningful progress without requiring terminal switching.
4. Support multiple concurrent agent sessions.
5. Provide a compact, notch-adjacent floating interface.
6. Preserve a useful activity history.
7. Operate locally by default.
8. Never silently approve agent permissions.
9. Remain useful when a particular agent integration is unavailable.
10. Provide a reliable fallback interface on Macs without a notch.

## 5. Non-Goals

The initial release will not:

- Replace the terminal or IDE.
- Replace the native permission interface of an agent.
- Automatically approve commands or permissions.
- Execute arbitrary commands received from agent events.
- Modify an agent's internal behavior to bypass its safeguards.
- Provide a cloud-based multi-user dashboard.
- Upload project activity or source code to a remote server.
- Depend on screenshots or OCR as the primary integration mechanism.
- Require a full embedded terminal emulator.
- Guarantee that every agent exposes identical lifecycle information.

## 6. Target Users

### Primary user

A software developer who uses AI coding agents from the terminal and often runs multiple sessions concurrently.

### Secondary users

- Developers working on long-running coding tasks.
- Developers using CLI agents alongside Xcode, VS Code, or other IDEs.
- Developers who want a lightweight overview of background agent activity.

## 7. Core User Stories

### Agent monitoring

- As a developer, I want to see which agents are running.
- As a developer, I want to know what each agent is currently doing.
- As a developer, I want to monitor multiple sessions independently.
- As a developer, I want to review activity that occurred while I was away.
- As a developer, I want to distinguish running, idle, waiting, completed, failed, and disconnected sessions.

### Permission notifications

- As a developer, I want to know immediately when an agent needs permission.
- As a developer, I want to identify the project and session associated with a request.
- As a developer, I want to inspect the requested action before deciding.
- As a developer, I want unresolved requests to remain visible until they are resolved.
- As a developer, I want to return to the originating terminal session.

### Desktop interaction

- As a developer, I want the compact pill to occupy minimal screen space.
- As a developer, I want the pill to expand when I hover over it.
- As a developer, I want a larger activity card when I click it.
- As a developer, I want the card to close without losing outstanding requests.
- As a developer, I want Comrad to remain accessible while using other applications.

### Privacy and control

- As a developer, I want agent activity to remain local by default.
- As a developer, I want control over notifications and retained history.
- As a developer, I do not want Comrad to approve permissions without my explicit action.

## 8. Functional Requirements

### FR-01: Agent discovery and registration

Comrad must support registering configured agent adapters and reporting their integration status.

### FR-02: Session tracking

Comrad must maintain separate state for every identifiable agent session.

### FR-03: Activity updates

Comrad must display supported progress, tool lifecycle, error, idle, and completion events.

### FR-04: Permission detection

Comrad must identify permission requests using documented integration signals.

### FR-05: Input detection

Comrad should identify explicit user-input requests when the agent exposes a reliable signal.

### FR-06: Notifications

Comrad must support macOS notifications for permission requests, user-input requests, and failures.

### FR-07: Compact floating interface

Comrad must provide a small, unobtrusive pill near the display notch where practical.

### FR-08: Hover expansion

The pill must expand when the pointer enters its interactive region.

### FR-09: Click expansion

Clicking the pill must open a larger activity card.

### FR-10: Request persistence

Outstanding attention requests must survive card dismissal and application restarts.

### FR-11: Activity history

Comrad must retain a configurable, bounded local history of agent events.

### FR-12: Session navigation

Comrad should help users return to the originating terminal or session when a reliable navigation method is available.

### FR-13: Integration health

Comrad must distinguish an active session from an integration that is disconnected or no longer producing reliable updates.

### FR-14: Configuration

Users must be able to configure supported integrations, notifications, startup behavior, and history retention.

### FR-15: Fallback interface

Users must be able to access activity through a menu bar entry point or standard window when the floating panel is unavailable.

## 9. User Experience Requirements

The application has three primary presentation states.

### Compact

- Small rounded pill.
- Comrad icon.
- Minimal activity indicator.
- Attention indicator when an unresolved request exists.
- No large text or persistent dashboard.

### Hover

- Smooth expansion.
- Summary of running agents.
- Short descriptions of current activity.
- Visible waiting or error indicators.
- No accidental permission approval.

### Expanded

- Agent and project identification.
- Current session status.
- Latest activity summary.
- Recent event timeline.
- Outstanding permission or input requests.
- Navigation to the originating session.
- Access to history and settings.

Clicking outside the expanded card or pressing Escape may close the card. Closing it must not resolve outstanding requests.

## 10. Agent Integration Requirements

### Claude Code

Use documented hooks and lifecycle events. Support permission-request detection and other progress events where the installed version exposes them.

### OpenCode

Use the documented plugin API and structured session, permission, and tool lifecycle events.

### Agy

Inspect the installed CLI's supported extension mechanisms. Implement a dedicated adapter using official events, plugins, SDKs, or a supported wrapper.

### Integration fallback

If an agent lacks a reliable event signal, Comrad must report the limitation and avoid presenting inferred states as confirmed facts.

## 11. Non-Functional Requirements

### Reliability

- Malformed events must not crash the app.
- Duplicate events must not create duplicate notifications.
- Unresolved requests must remain visible.
- Integration failures must be represented explicitly.
- App restarts must recover persisted state where possible.

### Performance

Initial engineering targets:
- Typical local event-to-UI update: under 250 ms.
- No unnecessary polling of inactive integrations.
- Minimal idle CPU and memory usage.
- Bounded event history and log sizes.

These are targets to validate through measurement, not guaranteed performance claims.

### Privacy

- Local-first operation.
- No telemetry by default.
- No unnecessary storage of complete conversations.
- Redaction of sensitive event fields.
- Explicit configuration changes for integrations.

### Accessibility

- Keyboard-accessible controls.
- VoiceOver-compatible labels.
- Adequate contrast.
- Reduced-motion support.
- Clear text labels for status indicators.

### Compatibility

Support a declared macOS deployment target and test on the supported macOS versions and display configurations.

## 12. MVP Scope

The first usable release includes:

1. Native SwiftUI application.
2. AppKit floating panel.
3. Compact, hover, and expanded states.
4. Simulated agent events.
5. Normalized event model.
6. Local event bridge.
7. Claude Code integration.
8. OpenCode integration.
9. Agy integration after capability verification.
10. macOS notifications.
11. Multiple session tracking.
12. Persistent unresolved requests.
13. Basic activity history.
14. Settings and integration health indicators.

## 13. Acceptance Criteria

The MVP is acceptable when:

- The app launches without a regular document window.
- The compact pill appears in the configured location.
- Hover and click transitions work reliably.
- Multiple sessions are displayed independently.
- A supported permission event creates an attention item.
- The corresponding notification is shown when enabled and authorized.
- Duplicate events do not create duplicate requests.
- Dismissing the card does not resolve a request.
- A matching resolution event clears the pending state.
- Session idle is not automatically treated as completion.
- Restarting Comrad does not silently discard unresolved requests.
- The app provides a fallback interface when notch placement is unavailable.
- No permission is approved automatically.
- Unsupported integration capabilities are disclosed.

## 14. Success Metrics

Evaluate the product using:

- Event-to-interface latency.
- Permission-event detection accuracy.
- Duplicate notification rate.
- Incorrect session-state transition rate.
- Missed attention requests.
- App crash rate.
- Idle CPU and memory consumption.
- Time required to identify the session requiring attention.

## 15. Risks and Dependencies

- Agent APIs may change.
- Different versions may expose different events.
- Not every agent exposes complete progress information.
- macOS floating-window behavior differs across displays and Spaces.
- Notification authorization may be denied.
- External terminal sessions may not be programmatically discoverable.
- Permission request resolution may not be exposed through public APIs.

## 16. Release Strategy

### Milestone 1
Floating UI and simulated events.

### Milestone 2
Normalized event pipeline and local persistence.

### Milestone 3
Claude Code integration.

### Milestone 4
OpenCode integration.

### Milestone 5
Agy integration and compatibility validation.

### Milestone 6
Reliability, accessibility, security, and distribution.

## 17. Product Principle

Comrad is an observer and supervisor, not an autonomous permission authority.

Its responsibility is to surface reliable information, preserve user control, and make it easy to return to the correct agent session.
