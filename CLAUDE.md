# CLAUDE.md - AI Assistant Guide for time-tracker

## Project Overview

**time-tracker** is a minimalist Pomodoro-style time tracking web application. It runs entirely in the browser with no backend, storing all data client-side. The project was "vibe coded" using Claude Artifacts and prioritizes simplicity over complexity.

## Codebase Structure

```
time-tracker/
├── webapp/
│   └── index.html    # Complete application (HTML + CSS + JS in one file)
├── LICENSE           # Apache 2.0 license
├── README.md         # Project description
└── CLAUDE.md         # This file
```

**Architecture:** Monolithic single-file application (1,069 lines)
- Lines 1-514: HTML structure and embedded CSS
- Lines 515-1067: Vanilla JavaScript application logic

## Technologies

- **HTML5** - Single-file application structure
- **CSS3** - Embedded styling (no external stylesheets)
- **Vanilla JavaScript (ES6+)** - No frameworks or libraries
- **Browser APIs:** Web Notifications, LocalStorage (potential), Fetch

**No build system, package manager, or external dependencies.**

## Key Files

### webapp/index.html

The entire application lives in this file:

**Main Views:**
- `#main-view` - Timer display with task description
- `#calendar-view` - Session history list

**Modals:**
- `#modal` - Task input to start session
- `#modify-modal` - Adjust running timer
- `#settings-modal` - Configure default duration
- `#reflection-modal` - Post-session survey

**Core State Variables:**
```javascript
let timerInterval = null;
let remainingSeconds = 0;
let isTimerRunning = false;
let completedSessions = [];
let currentSessionStart = null;
let defaultTimerSeconds = 25 * 60; // 25 minutes
let isCalendarView = false;
let currentSessionData = null;
```

**Key Functions:**
- `startTimer(task)` - Initialize and start timer
- `timerComplete()` - Handle timer completion
- `showReflectionModal()` - Post-session reflection UI
- `saveReflection()` - Store session data
- `renderSessions()` - Display completed sessions
- `formatTime(seconds)` - Format to HH:MM:SS or MM:SS

## Code Conventions

### Naming
- **HTML IDs:** kebab-case (`#task-input`, `#timer-display`)
- **CSS Classes:** kebab-case (`.productivity-btn`, `.session-item`)
- **JavaScript:** camelCase (`isTimerRunning`, `defaultTimerSeconds`)

### Styling
- **Theme:** Dark mode - black background (#000), white text (#fff)
- **Font:** "Comic Mono" monospace from Google Fonts
- **Colors:** Monochromatic (#000, #fff, #111, #222, #333)
- **Borders:** 2px solid #fff for interactive elements
- **Icons:** Emoji-based interface

### Keyboard Shortcuts
- `Cmd/Ctrl + Enter` - Start timer
- `Enter` - Confirm modals
- `Escape` - Cancel/close modals
- `E` - End timer early
- `Cmd/Ctrl + ,` - Open settings

### Session Data Model
```javascript
{
  task: string,
  startTime: Date,
  endTime: Date,
  accomplishments: string,
  productivity: 'happy' | 'meh' | 'sad',
  blockers: string
}
```

## Development Workflow

### Running the Application
Simply open `webapp/index.html` in any modern web browser. No build step required.

### Making Changes
1. Edit `webapp/index.html` directly
2. Refresh browser to see changes
3. Test keyboard shortcuts and all modals
4. Verify browser notifications work (requires permission)

### Git Workflow
- Use feature branches for development
- Create pull requests for merging
- Commit messages follow conventional format: `type(scope): description`

## Features

1. **Pomodoro Timer** - Default 25-minute countdown, customizable
2. **Task Tracking** - Name tasks before starting sessions
3. **Session History** - View completed sessions in calendar view
4. **Post-Session Reflection** - Rate productivity, note accomplishments/blockers
5. **Export to CSV** - Google Calendar compatible export
6. **Browser Notifications** - Alert when timer completes
7. **Clock Display** - Shows current time when timer inactive

## Common Tasks for AI Assistants

### Adding a New Feature
1. Identify the appropriate section in `webapp/index.html`
2. Add HTML elements within the correct view/modal
3. Add CSS styling following existing patterns (inline in `<style>` tag)
4. Add JavaScript logic following existing patterns (inline in `<script>` tag)
5. Test all keyboard shortcuts still work
6. Ensure dark theme consistency

### Modifying Timer Logic
- Timer functions are in the JavaScript section
- Core functions: `startTimer()`, `timerComplete()`, `updateClock()`
- State variables at top of script section

### Adding Persistence
- Currently sessions are lost on page refresh
- Use `localStorage` API for persistence
- Key candidates: `completedSessions`, `defaultTimerSeconds`

### Styling Changes
- All CSS is embedded in the `<style>` tag
- Follow existing dark theme conventions
- Use existing color palette (#000, #fff, #111, #222, #333)
- Maintain 2px white borders for interactive elements

## Important Considerations

### No Dependencies
This project intentionally has no npm packages or build tools. Keep it simple.

### Single File Architecture
All code stays in `webapp/index.html`. Don't split into multiple files unless specifically requested.

### Browser Compatibility
Target modern browsers (Chrome, Firefox, Safari, Edge). Uses ES6+ features.

### Accessibility
- Maintain keyboard navigation
- Keep font sizes readable
- Ensure sufficient contrast (white on black)

### Privacy
All data stays in the browser. No analytics, no backend, no external requests (except Google Fonts).

## Testing Checklist

When making changes, verify:
- [ ] Timer starts and counts down correctly
- [ ] All modals open and close properly
- [ ] Keyboard shortcuts work (Cmd/Ctrl+Enter, Escape, E)
- [ ] Sessions save to history after reflection
- [ ] Calendar view renders sessions correctly
- [ ] Export generates valid CSV
- [ ] Notifications fire on timer completion
- [ ] Time inputs auto-advance correctly
- [ ] Settings persist during session

## Known Limitations

1. **No persistence** - Sessions lost on page refresh
2. **No mobile optimization** - Some elements may overflow on small screens
3. **Global state** - All state in global scope
4. **No tests** - No automated testing framework

## Future Enhancement Ideas

- Add localStorage persistence for sessions
- Implement sound alerts option
- Add session statistics/analytics view
- Support for multiple timer presets
- Mobile-responsive design improvements
- Dark/light theme toggle
