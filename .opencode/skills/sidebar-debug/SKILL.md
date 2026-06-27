---
name: sidebar-debug
description: Debug and modify Opendian's auto-collapsing sidebar toolbars (the .nav-header / .nav-buttons-container that slide open on hover). Load this skill BEFORE editing §10 (auto-collapsing sidebar toolbars) or §11 (right sidebar / outline / backlinks) in theme.css, or when debugging sidebar layout issues caused by third-party plugin views.
---

# Opendian Sidebar Toolbar Debugging

Opendian's auto-collapsing sidebar toolbars (`§10 theme.css:1958-2089`) are
plagued by a recurring failure mode: rule-by-rule patch attempts that guess
at root causes without verifying them. This skill exists to prevent that.

## Source Of Truth

1. Do not edit sidebar rules from memory. Re-read the relevant section of
   `theme.css` first:
   - §10 auto-collapsing toolbar logic (~line 1958)
   - §11 right sidebar / outline / backlinks (~line 1951)
2. Before any rule change, inspect the live DOM in Obsidian DevTools:
   - `Cmd+Option+I` (mac) / `Ctrl+Shift+I` (win/linux) → Elements
   - Click the broken element and walk up the tree, comparing against a
     known-good view (left file-explorer or right outline)
3. For plugin views, the plugin's own `styles.css` (in
   `DEV_VAULT/.obsidian/plugins/<plugin>/`) is part of the cascade. Read it
   before assuming a CSS variable or padding is yours to control.
4. When debugging Git view specifically, the relevant templates are in
   `DEV_VAULT/.obsidian/plugins/obsidian-git/main.js` (search for
   `git-view`).

## Hard Rules (Read Before Editing)

1. **No rule-without-evidence.** If you cannot point to a DevTools
   observation that motivates a rule, do not write the rule. "I think this
   might fix it" is not evidence.
2. **Four-scenario walkthrough before every edit.** Mentally trace the
   proposed change against:
   - left file-explorer
   - right outline
   - right third-party plugin view (e.g. Git)
   - right backlinks / tags pane

   If any scenario's behavior is unclear, do not write the rule.
3. **Do not stack `!important` to win specificity.** If a rule you wrote
   does not appear to apply, the selector is wrong or specificity is not the
   issue. Diagnose before re-trying.
4. **Prefer scoped selectors over global ones.** A `[data-type="..."]`
   selector scoped to one plugin is always safer than a generic rule that
   affects every sidebar view.
5. **Do not change `position`, `padding`, or `margin` without first reading
   the full DOM of the affected view.** Layout changes cascade in
   surprising ways; one of them will break hover or focus behavior.

## What Worked (Verified Fixes)

### Git view: drag-handle pill drops below other rails

- **Symptom:** right pill (`nav-header::before`) is a few pixels lower than
  the left rail's when right sidebar shows Obsidian Git source control.
  Outline and backlinks align correctly.
- **Root cause:** DOM nesting depth. In a normal sidebar, `.nav-header`
  is a top-level child of `.workspace-leaf-content`, sitting **above**
  `.view-content` and not inheriting its padding. The Git plugin nests
  `.nav-header` **inside** `.view-content > main.git-view`, so it
  inherits the 10px top padding and the pill drops with it.
- **Fix:** zero the top padding for git-view only:
  ```css
  .workspace-split.mod-right-split
    .workspace-leaf-content[data-type="git-view"] .view-content {
    padding-top: 0 !important;
  }
  ```
  Specificity (0,5,0) + `!important` reliably wins the project's own
  (0,3,0) + `!important` baseline rule.
- **Why this fix is safe:** the bottom 8px padding from
  `theme.css:1953-1956` is untouched, so file lists keep their bottom
  breathing room. Other plugin views are unaffected because the
  `[data-type="git-view"]` selector is unique to Obsidian Git.

### Hover toolbar: second row clipped when toolbar overflows

- **Symptom:** when a plugin (e.g. Git) injects 8+ buttons into
  `.nav-buttons-container`, hover expands the toolbar but the second row
  is cut off at the bottom.
- **Root cause:** the project sets `max-height: 40px` on the
  hover-expanded container, sized for a single row.
- **Fix:** allow wrapping and raise the ceiling:
  ```css
  .workspace-split.mod-left-split .nav-buttons-container,
  .workspace-split.mod-right-split .nav-buttons-container {
    flex-wrap: wrap;  /* no-op for single-row toolbars */
  }
  .workspace-split.mod-left-split .nav-header:hover .nav-buttons-container,
  .workspace-split.mod-right-split .nav-header:hover .nav-buttons-container {
    max-height: 88px;  /* ceiling, not fixed height */
  }
  ```
- **Why this fix is safe:** `flex-wrap: wrap` is a no-op when content fits
  one line. `max-height` is a ceiling, so single-row toolbars stay
  ~32px tall and reveal identically.

## What Did Not Work (Do Not Retry)

These were attempted and reverted. They are recorded so future sessions
do not waste cycles re-deriving the same wrong turns.

- **Adding `flex-wrap: wrap` + `max-height: 88px` to *all* sidebars
  unconditionally.** Worked, but it left other sidebar views with too
  much empty space and was rejected on aesthetic grounds. The targeted
  version above is the correct shape.
- **Making `.nav-header` `position: absolute; top: 0` to "decouple" its
  vertical position from the parent's padding.** Took the layout apart.
  The expanded toolbar overlapped content below. The structural-coupling
  diagnosis (above) is the right approach.
- **Negative `margin-top` on `.nav-header` to cancel parent
  `padding-top`.** Worked visually but was rejected on grounds of
  fragility — the value depends on the parent's padding, which is
  owned by another rule. The fix that decouples cleanly is the Git
  view-specific one above.
- **`opacity: 0` + interactive fallback on `.nav-buttons-container` to
  mask the clipping issue.** Hack, not a fix; rejected.
- **Sibling selector `.nav-header + * { margin-top: 8px }` to push
  content below the strip down.** Did not apply to the right place in
  the DOM (the actual content is wrapped in an extra div the plugin
  inserts), so the rule hit an unrelated node and produced no visible
  effect. Do not retry without first reading the full DOM of the
  affected view.

## Validation Checklist

After any change to §10 or §11, before declaring done:

- [ ] Reload the theme in Obsidian (`Cmd+R` or toggle the theme off/on
      in Appearance settings)
- [ ] Visually compare drag-handle pill y-position across:
      - left file-explorer
      - right outline
      - right Git source control (open it explicitly)
      - right backlinks or tags pane
- [ ] Hover each sidebar header and confirm the toolbar opens with the
      correct visual behavior (no overlap, no clipping)
- [ ] Diff is scoped to §10 / §11; no incidental edits to unrelated
      rules

## Style Preferences (for sidebar code in particular)

- **One selector chain per concern.** If a rule tries to fix two things
  with the same selector, it is hiding a missed diagnosis.
- **No `!important` without a sibling rule that justifies the
  specificity.** Document *why* the `!important` is necessary
  (e.g. "wins project's own baseline rule").
- **Comment the structural cause, not the symptom.** Future sessions
  read the comment first; the structural reason is what they need.
