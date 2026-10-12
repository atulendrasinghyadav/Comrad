
# Comrad — UI/UX Design Specification

**Product:** Comrad  
**Platform:** macOS  
**Design Direction:** Minimal, native, dark, unobtrusive  
**Primary Interaction:** Compact pill → hover preview → expanded activity card  
**Version:** 1.0

---

## 1. Design Vision

Comrad should feel like a small, intelligent companion that lives on the macOS desktop.

Its interface should remain almost invisible when the developer is focused on other work, but immediately communicate when an AI agent requires attention.

The experience should prioritize:
- Minimal visual interruption.
- Fast recognition of agent status.
- Clear attention indicators.
- Smooth, predictable transitions.
- Native macOS interaction patterns.
- Consistent behavior across display configurations.
- Accessibility and reduced-motion support.

Comrad is not a conventional dashboard. The floating pill is the primary interaction surface, while the expanded card is the detailed control and inspection interface.

## 2. Design Principles

### 2.1 Quiet by default

The compact state should occupy minimal screen space and avoid unnecessary text.

### 2.2 Attention is meaningful

Use strong visual emphasis only for permission requests, explicit user input, and failures.

### 2.3 Information appears progressively

Show only essential status in the compact state. Reveal summaries on hover and detailed events on click.

### 2.4 State must be honest

Distinguish running, idle, waiting, completed, failed, and disconnected states.

### 2.5 Animation must preserve usability

Transitions should feel fluid without delaying access to important information.

### 2.6 Native behavior takes priority

Use SwiftUI and AppKit, respect system appearance, and avoid unnecessary custom window chrome.

## 3. Primary Interaction Model

Comrad has three primary visual states.

### State A: Compact

Purpose: remain visible without interrupting work.

Content:
- Comrad icon.
- Optional activity indicator.
- Attention indicator when required.

Behavior:
- Small rounded pill.
- Anchored near the notch when practical.
- No large labels.
- Click opens the expanded card.
- Hover begins the preview transition.

Suggested initial dimensions:
- Width: 72–100 points.
- Height: 32–44 points.
- Corner radius: approximately half the height.

Dimensions must be adjusted to the icon, notch geometry, and actual hardware.

### State B: Hover Preview

Purpose: provide a quick overview without requiring a click.

Content:
- Comrad icon.
- Running-agent count.
- Short activity summary.
- Waiting or failure indicators.
- Up to two concise session summaries when space permits.

Behavior:
- Expands smoothly.
- Remains readable at a glance.
- Does not take focus unnecessarily.
- Returns to compact when the pointer leaves, unless the expanded state is locked by a click.

Suggested initial dimensions:
- Width: 240–360 points.
- Height: 64–120 points.

The exact dimensions should adapt to content and display width.

### State C: Expanded Card

Purpose: expose actionable details and recent history.

Content:
- Application header.
- Running and waiting counts.
- Agent session rows.
- Project names.
- Current activity summaries.
- Outstanding requests.
- Recent activity timeline.
- Session navigation.
- History and settings entry points.

Behavior:
- Opens when the user clicks the pill.
- Remains expanded after pointer exit.
- Closes on outside click or Escape.
- Keeps outstanding requests pending after dismissal.
- Does not automatically approve or execute anything.

Suggested initial dimensions:
- Width: 340–440 points.
- Height: 240–480 points, subject to available screen space.

Use scrolling when content exceeds the available height.

## 4. Visual Design System

### 4.1 Color palette

Use semantic colors instead of scattering raw color values throughout the application.

Suggested dark appearance:

| Token | Suggested value | Usage |
|---|---|---|
| Surface | #17191E | Main card |
| Elevated surface | #22252C | Secondary surfaces |
| Border | #373B45 | Subtle outlines |
| Primary text | #F3F4F6 | Main content |
| Secondary text | #A8ADB8 | Descriptions |
| Active | #8DB8FF | Running and activity |
| Attention | #F0C674 | Permission or input needed |
| Success | #8BD5A7 | Confirmed completion |
| Error | #FF8585 | Confirmed failure |

These values are starting tokens, not mandatory fixed colors. Verify contrast and adapt to light and dark appearance.

Use system semantic colors where possible.

### 4.2 Status colors

Running:
- Use a subtle active indicator.
- Optional restrained progress animation.

Waiting:
- Use an attention indicator.
- Give permission or input requests clear text labels.

Completed:
- Use a success indicator.
- Avoid persistent celebratory animation.

Failed:
- Use an error indicator.
- Provide a concise explanation.

Disconnected:
- Use a neutral or warning indicator.
- Explain that current status may be stale.

Never rely on color alone to communicate a state.

## 5. Typography

Use native macOS system typography.

Suggested hierarchy:

| Element | Style |
|---|---|
| Card title | Semibold, 15–17 pt |
| Agent name | Medium, 13–15 pt |
| Activity summary | Regular, 12–14 pt |
| Metadata | Regular, 11–12 pt |
| Status label | Medium, 11–12 pt |
| Timestamps | Regular, 10–11 pt |

Prefer concise summaries and avoid truncating essential permission details.

Use line limits for routine summaries, but allow users to inspect full request details in the expanded card.

## 6. Shape and Spacing

Suggested design tokens:

- Compact pill corner radius: capsule.
- Card corner radius: 18–24 pt.
- Inner control corner radius: 8–12 pt.
- Base spacing unit: 4 pt.
- Compact horizontal padding: 12–16 pt.
- Card padding: 16–20 pt.
- Space between major sections: 12–16 pt.
- Space between agent rows: 8–12 pt.

Avoid excessive borders, heavy shadows, and multiple competing surface layers.

Use a subtle shadow when needed to distinguish the panel from bright backgrounds.

## 7. Iconography

Use SF Symbols where appropriate.

Suggested symbols:
- Comrad: application asset.
- Running: `terminal` or `gearshape.2`.
- Permission: `hand.raised` or `exclamationmark.shield`.
- Input required: `text.bubble`.
- Completed: `checkmark.circle.fill`.
- Failed: `xmark.circle.fill`.
- Disconnected: `wifi.slash`.
- History: `clock.arrow.circlepath`.
- Settings: `gearshape`.
- Open session: `arrow.up.right.square`.

Check symbol availability for the deployment target.

Do not use emoji as the primary production status icons.

## 8. Compact Pill Layout

Conceptual structure:

    ┌──────────────────────────┐
    │   [Comrad]  [Activity]   │
    └──────────────────────────┘

When no attention is required:
- Show the icon and a subtle activity signal.
- Avoid persistent text where possible.

When attention is required:
- Show a clear warning or notification indicator.
- Preserve the compact form.
- Do not force the entire card open without user interaction unless the user explicitly enables that behavior.

The pill must remain clickable and provide a sufficiently large hit target even when its visible appearance is compact.

## 9. Hover Preview Layout

Suggested structure:

    ┌──────────────────────────────────────┐
    │ Comrad                    2 running  │
    │                                      │
    │ Claude Code   Running tests          │
    │ OpenCode      Waiting for permission │
    └──────────────────────────────────────┘

Requirements:
- Limit the number of visible summaries.
- Prefer the most important attention item first.
- Use short, meaningful activity text.
- Keep the preview readable without scrolling.
- Avoid distracting progress animations.

If multiple agents are waiting, show an accurate count.

## 10. Expanded Card Layout

Suggested structure:

    ┌──────────────────────────────────────┐
    │ Comrad                 Settings      │
    │ 3 agents · 1 needs attention         │
    ├──────────────────────────────────────┤
    │ Claude Code                   Running│
    │ Implementing authentication tests    │
    │                                      │
    │ OpenCode            Permission needed│
    │ Review requested command             │
    │                                      │
    ├──────────────────────────────────────┤
    │ Recent activity                      │
    │ • Updated authentication tests        │
    │ • Started build                      │
    │ • Permission request received        │
    ├──────────────────────────────────────┤
    │ Open session       View history       │
    └──────────────────────────────────────┘

When a session requires attention, promote the relevant request to the top of the card.

### Permission request layout

Display:
- Agent name.
- Project name or path.
- Type of request.
- Concise description.
- Relevant command or operation when safely available.
- Request timestamp.
- Action to inspect the request.
- Action to open the originating session.

Comrad must not present an approval button unless a future implementation has an explicitly supported, verified permission-resolution API and a separately reviewed safety design.

## 11. Animation Specification

Use short, spring-based animations.

Suggested initial values:
- Hover expansion: 180–280 ms.
- Click expansion: 220–320 ms.
- Collapse: 150–220 ms.
- Status indicator: restrained and nonessential.

These are tuning targets rather than strict timing requirements.

### Transition rules

Compact → Hover:
- Expand width and height.
- Fade in summary text.
- Avoid sudden changes in position.

Hover → Compact:
- Fade out preview details.
- Return to the compact size.
- Avoid collapsing while the user is moving into the expanded content.

Hover → Expanded:
- Expand to the card layout.
- Reveal the activity timeline.
- Keep the card anchored consistently.

Expanded → Compact:
- Triggered by explicit dismissal.
- Preserve all unresolved requests.
- Do not discard event history.

### Reduced motion

Respect the system Reduce Motion setting.

When reduced motion is enabled:
- Reduce scale and spring animations.
- Prefer simple fades or immediate transitions.
- Avoid decorative continuous motion.

## 12. Positioning and Display Behavior

### Notch-equipped Mac

- Position the compact pill near the notch when feasible.
- Keep the pill visually distinct from the system interface.
- Anchor expansion to a consistent top-center position.
- Avoid obscuring essential menu bar controls.

### Mac without a notch

- Use a top-center floating pill where practical.
- Offer a menu bar fallback.
- Allow the user to choose a preferred position if supported.

### External displays

- Do not assume that an external monitor has a notch.
- Keep the floating panel on a predictable display.
- Recalculate placement when displays are connected or disconnected.

### Full-screen applications

- Test the panel's behavior in full-screen spaces.
- Avoid claiming universal visibility.
- Provide a fallback access path if the panel is unavailable.

## 13. Interaction Rules

- Clicking the pill toggles the expanded card.
- Hovering over the compact pill shows a preview.
- Moving into the preview must not immediately collapse it.
- Escape closes the expanded card.
- Clicking outside may close the expanded card.
- Closing the card does not acknowledge or resolve an agent request.
- Clicking an agent row selects that session.
- Opening a session uses a verified navigation mechanism.
- A missing navigation mechanism must produce a clear fallback.
- Destructive actions require confirmation.

## 14. Activity Timeline

Each event row should include:
- Event summary.
- Agent identity.
- Timestamp.
- Optional event category.
- Optional details on expansion.

The timeline should:
- Display newest relevant events first.
- Group repeated low-priority activity where practical.
- Preserve important permission and failure events.
- Avoid exposing unnecessary command output.
- Distinguish confirmed events from heuristic observations.

Routine progress must not produce system notifications by default.

## 15. Empty, Loading, and Error States

### No agents configured

Show:
- A concise explanation.
- Supported integrations.
- An action to configure an agent.

### No active sessions

Show:
- A quiet idle state.
- The most recent session summary if available.
- An action to view history.

### Integration unavailable

Show:
- Which integration is unavailable.
- The last known activity time.
- A useful diagnostic or reconnect action.

### No notification permission

Show:
- A clear explanation.
- A link or instruction to enable notifications in System Settings.

### Invalid event

Reject the event and record a sanitized diagnostic. Do not crash or display untrusted content as a system instruction.

## 16. Settings Design

The settings window should include:

### General
- Launch at login.
- Preferred display or positioning behavior.
- Default presentation behavior.
- Menu bar visibility.

### Agents
- Integration status.
- Installation status.
- Supported capabilities.
- Version compatibility.
- Install, configure, and uninstall actions.

### Notifications
- Permission request alerts.
- Input-required alerts.
- Failure alerts.
- Optional completion alerts.

### History and Privacy
- Retention period.
- Clear activity history.
- Local data location.
- Data collection disclosure.

### About
- App version.
- Supported agent versions.
- Diagnostics export, with sensitive values redacted.

## 17. Accessibility

- Use meaningful accessibility labels.
- Ensure controls are keyboard accessible.
- Support VoiceOver.
- Maintain readable text contrast.
- Do not use color as the sole state indicator.
- Support reduced motion.
- Ensure text scaling does not make requests inaccessible.
- Provide accessible names for icon-only buttons.
- Preserve clear focus behavior in the expanded card.

## 18. Implementation Mapping

### SwiftUI responsibilities

- Pill content.
- Agent rows.
- Attention cards.
- Timeline.
- Settings.
- Animations.
- Accessibility labels.

### AppKit responsibilities

- Floating panel lifecycle.
- Window level and activation behavior.
- Screen selection.
- Position calculations.
- Window dismissal integration.
- Display configuration changes.

### Domain responsibilities

- Session state.
- Request lifecycle.
- Event ordering.
- Status derivation.
- Attention prioritization.

### Integration responsibilities

- Agent-specific event handling.
- Session identification.
- Native permission events.
- Capability reporting.
- Integration health.

## 19. Design Acceptance Criteria

The design is accepted when:

- The compact pill remains unobtrusive.
- Hover expansion is predictable.
- Clicking produces a detailed card.
- The card does not collapse unexpectedly during interaction.
- Pending requests remain visible after dismissal.
- Multiple agents are distinguishable.
- Important attention states are visually obvious.
- Normal progress does not overwhelm the user.
- The interface works with supported display configurations.
- Reduced-motion and accessibility settings are respected.
- The application provides a fallback when notch positioning is unavailable.

## 20. Final Design Principle

Comrad should demand attention only when attention is useful.

Routine agent work belongs in the background. Permission requests, explicit questions, and failures must be easy to recognize, understand, and act on.
