# AI301 Phase I – Issue Selection

## Project
**GitVerse (Next.js)**
**Repository:** https://github.com/nisshchayarathi/gitverse-nextjs
**My Fork:** https://github.com/salabili212/gitverse-nextjs
**Selected Issue:** #530 – UI: DashboardLayout mobile sidebar missing Escape key to close
**Issue Link:** https://github.com/nisshchayarathi/gitverse-nextjs/issues/530


## Problem Summary
The DashboardLayout component's mobile sidebar can be dismissed by clicking
the backdrop overlay, but pressing the Escape key does nothing. This breaks
a standard accessibility expectation — users (especially keyboard users)
expect Escape to close overlays and dialogs. Fixing this brings the sidebar
in line with common UI conventions and improves keyboard accessibility.

## Why I Chose This Issue
**Skill match:** This issue uses React state management (`setMobileMenuOpen`)
and DOM event handling in a Next.js/TypeScript codebase — technologies I'm
comfortable with and want to get more practice using in a real, unfamiliar
repo rather than a personal project.

**Learning goal:** I want to practice reading someone else's component
structure, understanding existing state patterns before changing them, and
writing an accessible keyboard interaction correctly (attaching to the right
element, cleaning up listeners, avoiding conflicts with other key handlers).

**Understanding:** I've read `src/components/layout/DashboardLayout.tsx`
(lines 117–156). The sidebar currently closes only via `onClick` on the
backdrop overlay. My plan is to add an `onKeyDown` (or `keydown` effect)
that checks for the `Escape` key while the sidebar is open, and calls
`setMobileMenuOpen(false)`, matching the existing close pattern.

## Initial Plan
1. Fork and clone the repo, install dependencies, and run it locally.
2. Reproduce the bug: open the mobile sidebar, press Escape, confirm it does
   not close.
3. Read the surrounding component code to understand how the overlay and
   state are wired together.
4. Implement the Escape key handler, following the existing code style and
   the pattern in the issue's suggested fix.
5. Manually test: sidebar opens/closes via backdrop click (no regression),
   and now also closes via Escape.
6. Push the branch, open a pull request referencing #530, and respond to
   any maintainer feedback.

## Files Likely Involved
- `src/components/layout/DashboardLayout.tsx` (lines 117–156)
