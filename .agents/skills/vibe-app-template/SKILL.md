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
