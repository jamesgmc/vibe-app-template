---
name: vibe-app-template
description: Use this skill when asked to create a new "vibe application" or when the user wants to use the Vibe design system template.
---

# Vibe App Template Guidelines

The Vibe design system is a custom Vanilla HTML/CSS/JS template designed for sleek, dark-themed Electron desktop applications. It features glassmorphism, smooth animations, and a rich dark color palette.

## Template Location
The user clones this repository as the starting point for new vibe applications.
- `vibe-theme.css`: Contains the core CSS variables, layout grid, typography, and reusable components (buttons, modals, toasts, toggles).
- `index.html`: Contains the boilerplate HTML shell including the Electron drag titlebar, header layout, modal structures, and toast containers.

## How to use this template for new applications
When generating a new application for the user using this template:
1. **The Base Files are already here**: Because the user cloned the template repo as their starting point, `vibe-theme.css` and `index.html` are already present in the workspace root. You should modify them or add to them to build the application logic.
2. **Do Not Modify the Theme Directly**: For application-specific styles (like specific table layouts, unique widget colors, etc.), create a new `app.css` or `style.css` file and link it in the HTML *after* `vibe-theme.css`.
3. **Use the Pre-built Classes**:
   - **Buttons**: Use `<button class="btn primary">`, `<button class="btn danger">`, `<button class="btn warning">`, `<button class="btn transparent">`. For icon-only buttons, add `icon-only`.
   - **Badges**: Use `<span class="badge success">`, `badge warning`, `badge danger`, `badge primary`.
   - **Toggles**: Use the `.toggle-container` and `.switch` layout for on/off settings.
   - **Modals**: Duplicate the `#demoModal` structure for new overlays.
   - **Icons**: The template uses Lucide icons (`<i data-lucide="icon-name"></i>`). Make sure to call `lucide.createIcons()` after dynamically adding icons.
4. **Layout Structure**: Maintain the `.app-container` > `.header` + `.content` flex structure to ensure the custom Electron titlebar works correctly and the layout doesn't overflow unexpectedly.

Always prioritize building UI elements using the CSS variables defined in `:root` inside `vibe-theme.css` rather than hardcoding new colors, to maintain the "Vibe".

## Advanced Patterns: Administrator Elevation
If the user asks to implement an Administrator checking/elevation flow, use this standardized Vibe pattern:

1. **IPC Handlers in `main.js`**:
```javascript
// Check if currently running as admin
ipcMain.handle('check-elevation', async () => {
    return new Promise((resolve) => {
        require('child_process').exec('net session', (error) => { resolve(!error); });
    });
});

// Restart the application as admin
ipcMain.handle('relaunch-as-admin', async () => {
    return new Promise((resolve) => {
        const exe = process.execPath;
        const argsStr = process.argv.slice(1).map(arg => `'${arg}'`).join(',');
        const command = argsStr ? `Start-Process -FilePath '${exe}' -ArgumentList ${argsStr} -Verb RunAs` : `Start-Process -FilePath '${exe}' -Verb RunAs`;
        
        require('child_process').exec(`powershell -NoProfile -NonInteractive -Command "${command}"`, (error) => {
            if (!error) { app.quit(); resolve({ success: true }); } 
            else { resolve({ success: false }); }
        });
    });
});
```

2. **UI Implementation in `index.html`**:
Add these to the header to show the status:
```html
<span id="adminBadge" class="badge warning hidden">
    <i data-lucide="shield-check"></i> Administrator
</span>
<button id="restartAdminBtn" class="btn warning hidden">
    <i data-lucide="shield"></i> Run as Administrator
</button>
```

## Advanced Patterns: Danger Mode
For apps managing critical states, implement a "Danger Mode" toggle:
1. Add a toggle button in the header (only visible to admins).
2. When activated, reveal an "Actions" column in your tables that exposes destructive actions (like Delete, Disable).
3. Ensure UI changes smoothly without full reloads by manipulating local data arrays when actions succeed.

## Advanced Patterns: Expandable Table Rows
When dealing with data-heavy tables, prefer expandable rows over modals for simple metadata:
1. Add an `onclick` to your `<tr>` that dynamically creates and inserts a sibling `<tr>` right below it.
2. Style the expanded `<tr>` with a unique class (e.g. `.expanded-row`), `background-color: rgba(255, 255, 255, 0.02)`, and an inset box-shadow to indicate depth.
3. Include an "Expand All / Collapse All" toggle in the header that iterates through rows to expand them, being careful to update `lucide.createIcons()` after rendering.

## Advanced Patterns: Client-Side JSON Export
For applications that display tabular data, providing an export mechanism without a backend roundtrip is highly efficient:
1. Include an "Export JSON" button in the header that exports the currently filtered dataset.
2. Add a dedicated column containing an export button for individual rows.
3. Use a client-side Blob or Data URI approach to trigger the download directly via a hidden anchor tag.
```javascript
function exportData(dataObj, filename = 'export.json') {
    const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(dataObj, null, 2));
    const dlAnchorElem = document.createElement('a');
    dlAnchorElem.setAttribute("href", dataStr);
    dlAnchorElem.setAttribute("download", filename);
    document.body.appendChild(dlAnchorElem);
    dlAnchorElem.click();
    dlAnchorElem.remove();
}
```

## Advanced Patterns: Table Column Sorting
When tables contain a large number of rows, enable client-side column sorting:
1. Add state variables for `currentSortColumn` and `currentSortAsc`.
2. Wrap header text in clickable elements (`<span onclick="setSort('ColumnKey')">...</span>`) with an indicator icon (`<i data-lucide="arrow-up-down"></i>`).
3. Ensure the `th` element has an opaque background (e.g., `background: linear-gradient(rgba(255,255,255,0.02), rgba(255,255,255,0.02)), var(--bg-surface);`) so scrolling rows don't show through transparent headers.
4. Update your `filterTasks()` equivalent to `.sort()` the filtered array before rendering, correctly handling string normalization and numeric/date comparisons.

## Advanced Patterns: State Snapshots & Comparison
When the application needs to save the current state to disk and compare it later, use the Vibe Snapshot pattern:

1. **IPC Handlers in `main.js`**:
```javascript
const path = require('path');
const fs = require('fs');
const snapshotsDir = path.join(__dirname, 'snapshots');
if (!fs.existsSync(snapshotsDir)) fs.mkdirSync(snapshotsDir);

ipcMain.handle('save-snapshot', (event, description, data) => {
  const filename = `snapshot_${Date.now()}.json`;
  fs.writeFileSync(path.join(snapshotsDir, filename), JSON.stringify({
    id: filename, date: new Date().toISOString(), description, data
  }, null, 2));
  return { success: true, id: filename };
});

ipcMain.handle('get-snapshots', () => {
  if (!fs.existsSync(snapshotsDir)) return [];
  return fs.readdirSync(snapshotsDir)
    .filter(f => f.endsWith('.json'))
    .map(f => JSON.parse(fs.readFileSync(path.join(snapshotsDir, f), 'utf-8')))
    .sort((a, b) => new Date(b.date) - new Date(a.date));
});

ipcMain.handle('load-snapshot', (event, id) => {
  return JSON.parse(fs.readFileSync(path.join(snapshotsDir, id), 'utf-8'));
});
```

2. **UI Implementation in `index.html`**:
Add a comparison banner and a modal for snapshot management:
```html
<!-- Banner for Comparison Mode -->
<div id="comparisonBanner" class="comparison-banner hidden">
    <div class="compare-info">
        <i data-lucide="git-compare"></i> Comparing: Current vs Snapshot
    </div>
    <div class="compare-actions">
        <button id="exitCompareBtn" class="btn danger">Exit</button>
    </div>
</div>

<!-- Snapshots Modal -->
<div id="snapshotsModal" class="modal-overlay hidden">
    <div class="modal">
        <div class="modal-header">
            <h2>Manage Snapshots</h2>
            <button class="btn icon-only transparent" onclick="/* hide modal */"><i data-lucide="x"></i></button>
        </div>
        <div class="modal-body">
            <div class="snapshot-controls" style="display: flex; gap: 12px;">
                <input type="text" id="snapshotDescInput" placeholder="Description..." style="flex: 1;">
                <button id="takeSnapshotBtn" class="btn primary"><i data-lucide="camera"></i> Snapshot</button>
            </div>
            <div class="snapshot-list-container">
                <table class="snapshot-table" style="width: 100%;">
                    <thead><tr><th>Date</th><th>Description</th><th>Action</th></tr></thead>
                    <tbody id="snapshotsBody"></tbody>
                </table>
            </div>
        </div>
    </div>
</div>
```
