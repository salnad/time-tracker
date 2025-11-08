# Tauri Application Migration Plan

## Overview
This document outlines the plan to convert the current single-file HTML time tracking application into a native Tauri application for macOS (Apple Silicon).

## Current Application Analysis

**Current State:**
- Single-file HTML application (`webapp/index.html`) with embedded CSS and JavaScript
- **Core Features:**
  - Pomodoro timer (default 25 minutes, customizable)
  - Real-time clock display when not running a timer
  - Task entry before starting each session
  - Post-session reflection modal (accomplishments, productivity feeling, blockers)
  - Session history view
  - CSV export for Google Calendar import
  - Browser notifications on completion

**Key Keyboard Shortcuts:**
- `Cmd+Enter`: Start timer (when idle) or modify timer (when running)
- `Cmd+,`: Open settings
- `Enter`: Confirm various modals
- `Escape`: Cancel modals
- `E`: End session early (in modify modal)

---

## Phase 1: Tauri Project Setup

### 1.1 Install Prerequisites
**Action Items:**
- Ensure Rust is installed (via `rustup`)
- Verify Rust toolchain is up to date: `rustup update`
- Install Apple Silicon target: `rustup target add aarch64-apple-darwin`
- Install Tauri CLI: `cargo install tauri-cli` or `npm install -g @tauri-apps/cli`
- Verify Node.js is installed (for npm scripts)

**Verification:**
```bash
rustc --version
cargo --version
node --version
npm --version
```

### 1.2 Initialize Tauri Project
**Action Items:**
- Run `npm create tauri-app@latest` in a new directory or adjacent to current repo
- Configuration choices:
  - App name: `time-tracker`
  - Window title: `Time Tracker`
  - UI recipe: `Vanilla` (HTML/CSS/JS, no framework)
  - Package manager: `npm` (or your preference)

**Expected Structure:**
```
time-tracker/
├── src-tauri/
│   ├── src/
│   │   └── main.rs
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   └── build.rs
├── src/
│   ├── index.html
│   ├── styles.css
│   └── main.js
├── package.json
└── package-lock.json
```

### 1.3 Configure tauri.conf.json for macOS
**Action Items:**

1. **Set build target to macOS only:**
```json
{
  "build": {
    "beforeDevCommand": "",
    "beforeBuildCommand": "",
    "devPath": "../src",
    "distDir": "../src"
  },
  "bundle": {
    "active": true,
    "targets": ["dmg", "app"],
    "identifier": "com.timetracker.app",
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ],
    "macOS": {
      "minimumSystemVersion": "11.0"
    }
  }
}
```

2. **Configure window properties:**
```json
{
  "tauri": {
    "windows": [
      {
        "title": "Time Tracker",
        "width": 800,
        "height": 600,
        "minWidth": 600,
        "minHeight": 500,
        "resizable": true,
        "fullscreen": false,
        "center": true,
        "decorations": true,
        "transparent": false,
        "alwaysOnTop": false
      }
    ]
  }
}
```

3. **Set security configuration:**
```json
{
  "tauri": {
    "security": {
      "csp": null
    },
    "allowlist": {
      "all": false,
      "fs": {
        "all": false,
        "scope": ["$APPDATA/*"]
      },
      "dialog": {
        "all": false,
        "save": true
      },
      "notification": {
        "all": true
      }
    }
  }
}
```

**Verification:**
- Run `npm run tauri dev` to ensure project initializes correctly
- Verify window opens with default content
- Check that window is resizable and centered

---

## Phase 2: Core Application Migration

### 2.1 Migrate HTML Structure
**Action Items:**

1. **Copy content from `webapp/index.html` to `src/index.html`:**
   - Copy entire `<head>` section (lines 4-408)
   - Copy entire `<body>` section (lines 410-1068)
   - Keep DOCTYPE and html tag structure from Tauri template

2. **Handle external dependencies:**
   - Google Fonts dependency: `https://fonts.googleapis.com/css2?family=Comic+Mono:wght@400;700&display=swap`
   - **Options:**
     - Option A: Keep external font (requires network, simplest)
     - Option B: Download font files and host locally in `src/assets/fonts/`
   - **Recommended:** Start with Option A for quick setup, migrate to Option B later

**File: `src/index.html`**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Time Tracker</title>
    <link href="https://fonts.googleapis.com/css2?family=Comic+Mono:wght@400;700&display=swap" rel="stylesheet">
    <style>
        /* Copy entire style section from webapp/index.html (lines 9-407) */
    </style>
</head>
<body>
    <!-- Copy entire body content from webapp/index.html (lines 410-513) -->
    <script>
        /* Copy entire script section from webapp/index.html (lines 515-1067) */
    </script>
</body>
</html>
```

### 2.2 Verify Application Functionality
**Action Items:**

1. **Test core timer functionality:**
   - Run `npm run tauri dev`
   - Press `Cmd+Enter` to start a timer
   - Verify timer counts down correctly
   - Verify task display shows correctly
   - Test `Cmd+Enter` during timer to modify time
   - Test ending timer early with `E` key

2. **Test modal interactions:**
   - Test task input modal (Escape to cancel, Enter to confirm)
   - Test modify modal (all time inputs, navigation between fields)
   - Test settings modal (`Cmd+,` to open)
   - Test reflection modal after timer completes

3. **Test navigation:**
   - Click calendar icon (📅) to view sessions
   - Verify empty state shows when no sessions
   - Click home icon (🏠) to return

4. **Identify issues:**
   - Note any missing functionality
   - Check browser console for errors
   - Verify all keyboard shortcuts work

**Expected Issues:**
- Notifications may not work (need Tauri API integration)
- Sessions don't persist (in-memory only)
- Export functionality may need adjustment

### 2.3 Optional: Extract CSS and JavaScript
**Action Items (Optional - can defer to later):**

If you prefer separate files for maintainability:

1. **Extract CSS:**
   - Create `src/styles.css`
   - Move `<style>` content to this file
   - Add `<link rel="stylesheet" href="styles.css">` to `index.html`

2. **Extract JavaScript:**
   - Create `src/app.js`
   - Move `<script>` content to this file
   - Add `<script src="app.js"></script>` before `</body>` in `index.html`

**Recommended:** Keep everything inline initially for faster iteration, refactor later if needed.

---

## Phase 3: Replace Browser APIs with Tauri APIs

### 3.1 Add Required Tauri Dependencies
**Action Items:**

1. **Install Tauri API JavaScript package:**
```bash
npm install @tauri-apps/api
```

2. **Add Tauri plugins to `src-tauri/Cargo.toml`:**
```toml
[dependencies]
tauri = { version = "1.5", features = ["notification-all", "dialog-save", "fs-all"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

3. **Update `tauri.conf.json` allowlist:**
```json
{
  "tauri": {
    "allowlist": {
      "notification": {
        "all": true
      },
      "dialog": {
        "save": true
      },
      "fs": {
        "writeFile": true,
        "createDir": true,
        "readFile": true,
        "scope": ["$APPDATA/*"]
      }
    }
  }
}
```

### 3.2 Replace Web Notifications with Tauri Notifications
**Action Items:**

1. **Import Tauri notification API at top of script:**
```javascript
// Add to top of <script> section or app.js
import { sendNotification, isPermissionGranted, requestPermission } from '@tauri-apps/api/notification';
```

**Note:** For inline script, use dynamic import:
```javascript
let tauriNotification;
(async () => {
    tauriNotification = await import('@tauri-apps/api/notification');
})();
```

2. **Replace notification permission request (lines 557-560):**

**Old code:**
```javascript
// Request notification permission on load
if ('Notification' in window && Notification.permission === 'default') {
    Notification.requestPermission();
}
```

**New code:**
```javascript
// Request notification permission on load
(async () => {
    let permissionGranted = await tauriNotification.isPermissionGranted();
    if (!permissionGranted) {
        const permission = await tauriNotification.requestPermission();
        permissionGranted = permission === 'granted';
    }
})();
```

3. **Replace notification sending (lines 632-638):**

**Old code:**
```javascript
// Show notification
if ('Notification' in window && Notification.permission === 'granted') {
    new Notification('Pomodoro Complete!', {
        body: 'Time for a break! You completed: ' + taskDisplay.textContent,
        icon: 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><text y="75" font-size="75">🍅</text></svg>'
    });
}
```

**New code:**
```javascript
// Show notification
(async () => {
    let permissionGranted = await tauriNotification.isPermissionGranted();
    if (permissionGranted) {
        await tauriNotification.sendNotification({
            title: 'Pomodoro Complete!',
            body: 'Time for a break! You completed: ' + taskDisplay.textContent
        });
    }
})();
```

**Verification:**
- Start a 5-second timer for testing
- Verify notification appears when timer completes
- Check that notification shows even when app is not focused

### 3.3 Implement Session Persistence with Tauri Filesystem
**Action Items:**

1. **Import Tauri filesystem APIs:**
```javascript
import { appDataDir, createDir } from '@tauri-apps/api/path';
import { readTextFile, writeTextFile } from '@tauri-apps/api/fs';
```

For inline script:
```javascript
let tauriPath, tauriFs;
(async () => {
    tauriPath = await import('@tauri-apps/api/path');
    tauriFs = await import('@tauri-apps/api/fs');
})();
```

2. **Create save function:**
```javascript
async function saveSessions() {
    try {
        const dataDir = await tauriPath.appDataDir();
        const filePath = dataDir + 'sessions.json';

        // Convert dates to ISO strings for JSON serialization
        const sessionsToSave = completedSessions.map(session => ({
            ...session,
            startTime: session.startTime.toISOString(),
            endTime: session.endTime.toISOString()
        }));

        await tauriFs.writeTextFile(filePath, JSON.stringify(sessionsToSave, null, 2));
        console.log('Sessions saved successfully');
    } catch (error) {
        console.error('Failed to save sessions:', error);
    }
}
```

3. **Create load function:**
```javascript
async function loadSessions() {
    try {
        const dataDir = await tauriPath.appDataDir();
        const filePath = dataDir + 'sessions.json';

        const content = await tauriFs.readTextFile(filePath);
        const loadedSessions = JSON.parse(content);

        // Convert ISO strings back to Date objects
        completedSessions = loadedSessions.map(session => ({
            ...session,
            startTime: new Date(session.startTime),
            endTime: new Date(session.endTime)
        }));

        console.log(`Loaded ${completedSessions.length} sessions`);
    } catch (error) {
        console.log('No existing sessions file or failed to load:', error);
        completedSessions = [];
    }
}
```

4. **Call loadSessions on app start:**
```javascript
// Add near top of script, after variable declarations
(async () => {
    await loadSessions();
})();
```

5. **Call saveSessions after completing a session:**

Modify the `saveReflection()` function (line 1039):
```javascript
function saveReflection() {
    currentSessionData.accomplishments = accomplishmentsInput.value.trim();
    currentSessionData.blockers = blockersInput.value.trim();

    completedSessions.push(currentSessionData);
    currentSessionData = null;

    // Save to disk
    saveSessions();

    reflectionModal.classList.remove('active');

    // Reset UI
    taskDisplay.textContent = '';
    statusEl.textContent = '';
    instructions.style.display = 'block';
    updateClock();
}
```

**Verification:**
- Complete a pomodoro session with reflection
- Close and restart the app
- Navigate to sessions view (📅)
- Verify saved session appears

### 3.4 Replace CSV Export with Tauri Save Dialog
**Action Items:**

1. **Import Tauri dialog API:**
```javascript
import { save } from '@tauri-apps/api/dialog';
```

For inline script:
```javascript
let tauriDialog;
(async () => {
    tauriDialog = await import('@tauri-apps/api/dialog');
})();
```

2. **Replace export button handler (lines 882-914):**

**Old code:**
```javascript
calendarExportBtn.addEventListener('click', () => {
    if (completedSessions.length === 0) {
        alert('No completed sessions to export!');
        return;
    }

    let csvContent = 'Subject,Start Date,Start Time,End Date,End Time,Description\n';

    completedSessions.forEach(session => {
        const startDate = session.startTime.toLocaleDateString('en-US');
        const startTime = session.startTime.toLocaleTimeString('en-US', { hour12: false });
        const endDate = session.endTime.toLocaleDateString('en-US');
        const endTime = session.endTime.toLocaleTimeString('en-US', { hour12: false });

        const taskName = session.task.replace(/"/g, '""');

        csvContent += `"${taskName}",${startDate},${startTime},${endDate},${endTime},Pomodoro session\n`;
    });

    const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
    const link = document.createElement('a');
    const url = URL.createObjectURL(blob);

    link.setAttribute('href', url);
    link.setAttribute('download', `pomodoro_sessions_${new Date().toISOString().split('T')[0]}.csv`);
    link.style.visibility = 'hidden';

    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);

    alert(`Exported ${completedSessions.length} session(s)! Import this CSV into Google Calendar.`);
});
```

**New code:**
```javascript
calendarExportBtn.addEventListener('click', async () => {
    if (completedSessions.length === 0) {
        alert('No completed sessions to export!');
        return;
    }

    // Build CSV content
    let csvContent = 'Subject,Start Date,Start Time,End Date,End Time,Description\n';

    completedSessions.forEach(session => {
        const startDate = session.startTime.toLocaleDateString('en-US');
        const startTime = session.startTime.toLocaleTimeString('en-US', { hour12: false });
        const endDate = session.endTime.toLocaleDateString('en-US');
        const endTime = session.endTime.toLocaleTimeString('en-US', { hour12: false });

        const taskName = session.task.replace(/"/g, '""');

        csvContent += `"${taskName}",${startDate},${startTime},${endDate},${endTime},Pomodoro session\n`;
    });

    // Show save dialog
    const defaultFilename = `pomodoro_sessions_${new Date().toISOString().split('T')[0]}.csv`;
    const filePath = await tauriDialog.save({
        defaultPath: defaultFilename,
        filters: [{
            name: 'CSV',
            extensions: ['csv']
        }]
    });

    if (filePath) {
        try {
            await tauriFs.writeTextFile(filePath, csvContent);
            alert(`Exported ${completedSessions.length} session(s) to ${filePath}! Import this CSV into Google Calendar.`);
        } catch (error) {
            alert('Failed to save file: ' + error);
        }
    }
});
```

**Verification:**
- Complete at least one session
- Navigate to sessions view
- Click "Export to Google Calendar"
- Verify save dialog appears
- Choose a location and save
- Verify file is created with correct CSV content
- Test importing into Google Calendar

### 3.5 Handle Module Imports Properly
**Action Items:**

Since we're using inline scripts in HTML, we need to handle Tauri API imports carefully.

**Option A: Keep inline with dynamic imports (Simpler)**
- Use the `(async () => { ... })()` pattern shown above
- Wrap all Tauri API calls in async functions
- Keep all code in `<script>` tag

**Option B: Use ES modules (Cleaner, recommended)**
1. Extract JavaScript to `src/app.js`
2. Update `index.html`:
```html
<script type="module" src="app.js"></script>
```
3. Use regular imports in `app.js`:
```javascript
import { sendNotification } from '@tauri-apps/api/notification';
import { appDataDir } from '@tauri-apps/api/path';
import { readTextFile, writeTextFile } from '@tauri-apps/api/fs';
import { save } from '@tauri-apps/api/dialog';
```

**Recommended:** Use Option B (ES modules) as it's cleaner and more maintainable.

---

## Phase 4: Keyboard Shortcuts Enhancement

### High-Level Overview

**Current State:**
- All existing keyboard shortcuts work via JavaScript event listeners
- Shortcuts only work when app is focused

**Goals:**
- Preserve all existing in-app shortcuts
- Add foundation for global shortcuts (system-wide)

**Implementation Approach:**
- Use `tauri-plugin-global-shortcut` for system-wide hotkeys
- Register shortcuts like `Cmd+Shift+T` to focus app from anywhere
- Keep architecture simple but extensible

**Future Enhancements:**
- Quick-start timer without focusing app
- Status display in menu bar
- Global pause/resume

---

## Phase 5: macOS-Specific Enhancements

### High-Level Overview

**Goals:**
- Native macOS app feel
- Integrate with macOS system features
- Proper lifecycle management

**Key Features:**
- Menu bar integration (optional)
- Dock icon with timer status badge
- Hide-on-close behavior (don't quit)
- Timer state restoration after crash

**Implementation Approach:**
- Configure Tauri window behaviors in `tauri.conf.json`
- Add macOS bundle settings
- Implement state save/restore on app lifecycle events

---

## Phase 6: Development & Build Setup

### High-Level Overview

**Development Workflow:**
- Use `npm run tauri dev` for hot-reload development
- Test on Apple Silicon Mac continuously
- Verify all keyboard shortcuts work natively

**Build Process:**
- Run `npm run tauri build` for production builds
- Generates `.app` bundle in `src-tauri/target/release/bundle/macos/`
- Code signing optional for personal use
- Can create `.dmg` installer if desired

**Build Configuration:**
- Optimize for Apple Silicon (aarch64-apple-darwin)
- Configure bundle identifier and version
- Set up app icons (can use emoji initially)

---

## Future Phases (Not in Initial Scope)

### Phase 7: Data Persistence & Export
- Enhanced JSON schema with versioning
- Backup/restore functionality
- Export to multiple formats (CSV, JSON, iCal)

### Phase 8: Global Shortcuts
- System-wide hotkeys
- Quick-start from anywhere
- Status in menu bar

### Phase 9: UI Enhancements
- Always-on-top mode
- Compact/mini mode
- Customizable themes
- Window transparency options

---

## Success Criteria

After completing Phases 1-6, you should have:
1. ✅ A native macOS `.app` that runs on Apple Silicon
2. ✅ All existing features working (timer, sessions, export)
3. ✅ All keyboard shortcuts preserved and functional
4. ✅ Sessions persist across app restarts
5. ✅ Native notifications that work when app is unfocused
6. ✅ Native save dialog for CSV export
7. ✅ Clean development workflow for future enhancements

---

## Notes & Considerations

### Why Tauri?
- **Small binary size** (~3-5MB vs Electron's ~100MB+)
- **Native performance** (Rust + system WebView)
- **Low memory footprint**
- **Native OS integration** (notifications, dialogs, etc.)
- **Apple Silicon native** without extra configuration

### Risks & Mitigations
- **Risk:** Tauri API learning curve
  - **Mitigation:** Start with simple features, iterate
- **Risk:** Breaking existing keyboard shortcuts
  - **Mitigation:** Test thoroughly in Phase 2
- **Risk:** Notification permissions issues on macOS
  - **Mitigation:** Test early, handle gracefully

### Development Tips
- Test frequently with `npm run tauri dev`
- Keep browser dev tools open for debugging
- Use `console.log` liberally during migration
- Commit after each phase completes successfully
