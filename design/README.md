# design/: canonical UI source files

Single source of truth for the shared look and behavior (see STANDARDS.md
§1). Each app vendors verbatim copies so the sites stay build-free and work
offline; a CI drift check diffs the vendored files against the pinned
`actions` tag and fails on divergence.

## Files

- `tokens.css` — `:root` light tokens + `.dark-mode` overrides. Exact copy
  of the `light_slides` `:root` block; semantic app accents are defined
  app-side.
- `dialogs.css` — pills, theme toggle, help/settings overlay, card, list,
  `kbd`. Requires `tokens.css`.
- `dialogs.js` — `initTheme(toggleParentSelector)` and
  `wireDialog(dialogId, buttonId)`. Keyboard shortcuts and input guards stay
  app-side (they differ per app).

## Adopting in an app

1. Copy the three files (or the CSS relevant to the app) into the app's
   static dir, or paste the blocks into the existing stylesheet.
2. Give the help overlay `id="help"` (and settings `id="settings"`), the
   card `class="help-card"`, and wire with `wireDialog("help", "help_btn")`.
3. Call `initTheme("<actions container selector>")` once at startup.
4. List app shortcuts in the help card's `dl.help-list`; document only
   implemented keys.
