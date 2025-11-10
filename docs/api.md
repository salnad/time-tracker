| Pomodoro Timer Web App Documentation |
| --- |

# Overview
- Single-page Pomodoro timer that runs entirely in the browser (no external state except `Notification` API permission).
- Core UI is created declaratively in `webapp/index.html`; all logic lives inside the embedded `<script>`.
- The script exposes its functions and state on the global scope, so they can be reused or extended from the DevTools console or additional scripts.

# Application Architecture
- **User Interface**: Static HTML structure composed of primary views (`#main-view`, `#calendar-view`) and modal dialogs for task entry, timer modification, settings, and reflection.
- **State Management**: Plain JavaScript variables track timer progress, session history, and modal state. No external storage is used.
- **Event Handling**: DOM event listeners wire up keyboard shortcuts, button actions, and input navigation to the core functions documented below.

# Data Model
`completedSessions` stores session records. Each record conforms to:

| Field | Type | Description |
| --- | --- | --- |
| `task` | `string` | Task description provided when the Pomodoro starts. |
| `startTime` | `Date` | Timestamp captured when the timer begins. |
| `endTime` | `Date` | Timestamp captured when the timer completes or ends early. |
| `accomplishments` | `string` | Reflection text captured at the end of the session (optional). |
| `productivity` | `"happy" \| "meh" \| "sad" \| null` | Productivity feeling selected in the reflection modal (optional). |
| `blockers` | `string` | Reflection text describing blockers (optional). |

`currentSessionData` temporarily holds the active session record between timer completion and reflection submission.

# UI Components
| Selector | Element | Purpose |
| --- | --- | --- |
| `#display` | `div` | Shows the live clock (idle) or countdown (active timer). |
| `#status` | `div` | Displays status messages such as focus prompts and completion alerts. |
| `#task-display` | `div` | Mirrors the active task name while a timer runs. |
| `#instructions` | `div` | Explains the primary keyboard shortcut for starting a Pomodoro. |
| `#modal` | `div` | Start modal for entering a task name. |
| `#task-input` | `input` | Text field inside the start modal. |
| `#modify-modal` | `div` | Timer modification modal shown while a Pomodoro is running. |
| `#hours-input`, `#minutes-input`, `#seconds-input` | `input` | Numeric inputs for adjusting the remaining timer duration. |
| `#settings-modal` | `div` | Modal for configuring the default Pomodoro duration. |
| `#settings-hours-input`, `#settings-minutes-input`, `#settings-seconds-input` | `input` | Numeric inputs for default duration settings. |
| `#main-view` | `section` | Primary view that shows the clock and status. |
| `#calendar-view` | `section` | Secondary view listing completed sessions with export capabilities. |
| `#sessions-list` | `div` | Container populated by `renderSessions()`. |
| `#calendar-export-btn` | `button` | Triggers CSV export for Google Calendar imports. |
| `#reflection-modal` | `div` | Post-session modal for journaling accomplishments and blockers. |
| `.productivity-btn` | `button` | Emoji buttons capturing the perceived productivity level. |
| `.top-right-buttons #nav-btn` | `button` | Toggles between main and calendar views. |
| `.top-right-buttons #settings-btn` | `button` | Opens the settings modal. |

# Global State and Configuration
- `timerInterval: number | null` – Identifier returned by `setInterval`; cleared when the timer stops.
- `remainingSeconds: number` – Countdown value updated every second while the timer runs.
- `isTimerRunning: boolean` – Signals whether the timer is active.
- `completedSessions: SessionRecord[]` – Array of completed session objects.
- `currentSessionStart: Date | null` – Timestamp captured when the active session begins.
- `defaultTimerSeconds: number` – Default duration in seconds (initially `25 * 60`).
- `isCalendarView: boolean` – Tracks whether the calendar view is currently visible.
- `currentSessionData: SessionRecord | null` – Temporary storage for the just-completed session awaiting reflection input.

# Public Functions
All functions below are declared in the global scope and can be invoked directly (e.g., from the browser console or additional scripts).

## `updateClock(): void`
- **Purpose**: Refreshes `#display` with the current time when no timer is running.
- **Side Effects**: Updates the DOM; no return value.
- **Usage**:
  ```javascript
  // Force the idle clock to refresh immediately
  updateClock();
  ```

## `formatTime(seconds: number): string`
- **Purpose**: Converts a second-based duration to `HH:MM:SS` (if hours > 0) or `MM:SS`.
- **Parameters**:
  - `seconds`: Total duration in seconds.
- **Returns**: Formatted time string.
- **Usage**:
  ```javascript
  const label = formatTime(90); // "01:30"
  ```

## `startTimer(task: string): void`
- **Purpose**: Begins a Pomodoro for `task`, resets the countdown to `defaultTimerSeconds`, and starts ticking.
- **Parameters**:
  - `task`: Description shown in `#task-display`.
- **Side Effects**:
  - Populates UI elements (`#display`, `#status`, `#task-display`).
  - Hides the instructional text.
  - Creates an interval that decrements `remainingSeconds` each second.
  - Captures `currentSessionStart`.
- **Usage**:
  ```javascript
  startTimer('Outline documentation');
  ```
- **Notes**: Safe to call only when no timer is running; otherwise use `showModifyModal()` to adjust the active timer.

## `timerComplete(): void`
- **Purpose**: Finalizes an active Pomodoro session when the countdown reaches zero or ends early.
- **Side Effects**:
  - Stops the timer interval and resets countdown state.
  - Captures session metadata in `currentSessionData`.
  - Updates status UI and triggers a browser notification (if allowed).
  - Opens the reflection modal after a short delay via `showReflectionModal()`.
- **Usage**:
  ```javascript
  // Immediately mark the current timer as complete
  timerComplete();
  ```

## `showStartModal(): void`
- **Purpose**: Displays the task input modal and focuses the input field.
- **Usage**:
  ```javascript
  showStartModal();
  ```
- **Notes**: Automatically invoked by the `Cmd/Ctrl + Enter` shortcut when the timer is idle.

## `showModifyModal(): void`
- **Purpose**: Opens the modify-timer modal pre-filled with the remaining duration.
- **Side Effects**: Sets focus on `#hours-input` and selects its contents.
- **Usage**:
  ```javascript
  if (isTimerRunning) {
    showModifyModal();
  }
  ```
- **Notes**: Triggered by `Cmd/Ctrl + Enter` while a Pomodoro is active.

## `renderSessions(): void`
- **Purpose**: Populates `#sessions-list` with cards summarizing each completed session.
- **Side Effects**:
  - Clears `#sessions-list` content.
  - Sorts sessions by start time (descending).
  - Injects reflection details when available.
- **Usage**:
  ```javascript
  // After adding custom session data:
  completedSessions.push(...customSessions);
  renderSessions();
  ```

## `showReflectionModal(): void`
- **Purpose**: Resets reflection inputs and displays the post-session modal.
- **Side Effects**: Clears textareas, deselects productivity buttons, focuses the accomplishments field.
- **Usage**:
  ```javascript
  showReflectionModal();
  ```
- **Notes**: Called automatically from `timerComplete()`.

## `saveReflection(): void`
- **Purpose**: Commits reflection data to the most recent `currentSessionData`, appends it to `completedSessions`, and resets the UI to idle.
- **Side Effects**:
  - Pushes a session record into `completedSessions`.
  - Hides the reflection modal and restores idle UI state.
  - Invokes `updateClock()` to resume the live clock.
- **Usage**:
  ```javascript
  accomplishmentsInput.value = 'Wrote documentation outline';
  blockersInput.value = 'None';
  currentSessionData.productivity = 'happy';
  saveReflection();
  ```

## `setupTimeInputNavigation(): void`
- **Purpose**: Configures arrow-key and auto-advance behavior for the modify-timer inputs.
- **Usage**: Executed once during initialization; safe to call again after injecting new inputs.

## `setupSettingsTimeInputNavigation(): void`
- **Purpose**: Mirrors `setupTimeInputNavigation()` for the settings modal inputs.
- **Usage**: Executed once during initialization; can be rerun if the settings modal is re-rendered.

# Event Hooks and Shortcuts
- **Keyboard Shortcuts**:
  - `Cmd/Ctrl + Enter`: When idle, opens the start modal; when timing, opens the modify modal.
  - `Cmd/Ctrl + ,`: Opens the settings modal.
  - `Escape`: Closes whichever modal (start, modify, settings, reflection) is open.
  - `Enter` (inside modify modal): Applies the new duration.
  - `E` (inside modify modal): Ends the timer early.
  - `Cmd/Ctrl + Enter` (inside reflection modal): Saves the reflection.
- **Button Events**:
  - `#change-timer-btn`: Applies new countdown values from modify modal inputs.
  - `#end-btn`: Ends the timer and triggers `timerComplete()`.
  - `#calendar-export-btn`: Generates a CSV of `completedSessions` and prompts download.
  - `.productivity-btn`: Sets `currentSessionData.productivity` and updates selection styles.
- **Input Events**: Numeric inputs enforce bounds (`23:59:59` max) and auto-advance on valid data entry, implemented in the `setup*Navigation()` helpers.

# Usage Recipes
Use the browser console (or inject additional scripts) to compose the behaviors below.

## Start a Pomodoro with a Custom Duration
```javascript
defaultTimerSeconds = 45 * 60;
startTimer('Deep work session');
```
> Tip: If you change `defaultTimerSeconds` while idle, call `updateClock()` to refresh the display.

## Fast-Forward to Reflection
```javascript
if (isTimerRunning) {
  timerComplete();
}
```

## Log a Manual Session
```javascript
completedSessions.push({
  task: 'Team sync',
  startTime: new Date('2025-11-10T09:00:00'),
  endTime: new Date('2025-11-10T09:25:00'),
  accomplishments: 'Discussed roadmap',
  productivity: 'happy',
  blockers: ''
});
renderSessions();
```

## Export Sessions Programmatically
```javascript
// Ensure there is at least one completed session first
calendarExportBtn.click();
```

# Extensibility Notes
- All state is in-memory; to persist sessions across reloads, hook into `saveReflection()` to serialize `completedSessions` (e.g., to `localStorage`) and hydrate it before `renderSessions()`.
- To customize notifications, update the `Notification` payload in `timerComplete()`.
- For alternate timer lengths (e.g., short/long breaks), consider wrapping `startTimer()` to set `defaultTimerSeconds` before invocation and restore it afterwards.
