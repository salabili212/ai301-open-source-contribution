# AI301 Open Source Contribution – Suliman Alabili

## Phase I — Issue Selection

### Project

**GitVerse (Next.js)**
**Repository:** https://github.com/nisshchayarathi/gitverse-nextjs
**My Fork:** https://github.com/salabili212/gitverse-nextjs
**Selected Issue:** [#530 – UI: DashboardLayout mobile sidebar missing Escape key to close](https://github.com/nisshchayarathi/gitverse-nextjs/issues/530)
**My Comment on Issue:** https://github.com/nisshchayarathi/gitverse-nextjs/issues/530#issuecomment-5087508292

### Why I selected this issue

I selected this issue because it is a well-scoped, self-contained UI accessibility bug in a real Next.js codebase. The problem is clearly described: the mobile sidebar overlay has no keyboard dismiss path, which breaks standard accessibility expectations for dismissible overlays. The fix touches exactly one file and follows a well-known React pattern (`useEffect` + `document.addEventListener`), making it realistic to complete and submit within the course timeline.

It also gave me the opportunity to read an unfamiliar codebase, understand how the component's state is managed, and contribute something that has real impact for keyboard and screen reader users.

### Initial Plan

1. Fork the repository and set up the local dev environment.
2. Reproduce the missing behavior by inspecting the component source.
3. Study how the sidebar state (`mobileMenuOpen`) is already managed.
4. Add a `useEffect` to attach an Escape key listener scoped to when the sidebar is open.
5. Verify no other files need to change and run lint/type checks.
6. Open a pull request and respond to any maintainer feedback.

---

## Phase II — Reproduce and Plan

### Environment Setup

Hit a `Server Error: Missing required environment variables: INTERNAL_WORKER_SECRET` on first run. Inspected `lib/env.ts` and found the app validates `DATABASE_URL`, `NEXTAUTH_URL`, `GEMINI_API_KEY`, and `INTERNAL_WORKER_SECRET` on boot, and requires either `TOKEN_ENCRYPTION_KEY` or `KMS_KEY_ID`.

**Challenge:** No local database or auth backend was available, and standing one up was out of scope for a small UI fix.

**Resolution:** The project has a built-in validation skip for CI environments. Used that flag to boot the dev server without a live database:

```powershell
$env:CI="true"; npm run dev
```

This got the app running on `localhost:3001`. Full authenticated pages (like `/Dashboard`) still require a real database connection to render — confirmed this when a signup attempt returned a backend error instead of a valid response. Reproduction of the actual bug was therefore done at the code level (see below), since the component and its bug are fully visible in the source without needing a live session.

**Branch:** `fix/sidebar-escape-key` (pushed to my fork). Note: this doesn't follow the `fix-issue-NNN` numbering convention exactly — for future issues I'll use `fix-issue-530`-style naming from the start.

### Reproduction Steps

1. Clone the fork and run `npm install`.
2. Set `CI=true` and run `npm run dev` to bypass env validation.
3. Open `src/components/layout/DashboardLayout.tsx` and locate the mobile sidebar block (originally lines 117–156).
4. Search the file for any keyboard event handling:

```powershell
Select-String -Path src\components\layout\DashboardLayout.tsx -Pattern "keydown|Escape|onKeyDown"
```

5. Observe the result: zero matches. The only dismiss handler present is `onClick={() => setMobileMenuOpen(false)}` on the backdrop `<div>`.

**Expected behavior:** Pressing Escape while the mobile sidebar is open should close it, consistent with standard behavior for dismissible overlays/dialogs.

**Actual behavior:** Pressing Escape does nothing. The sidebar only closes via a mouse click on the backdrop overlay; there is no keyboard-triggered close path at all.

### Plan (UMPIRE)

**Understand:** The mobile sidebar (`DashboardLayout.tsx`) opens as an overlay controlled by `mobileMenuOpen` state. It currently closes only through a single `onClick` handler on the backdrop `<div>`. There is no keyboard accessibility path to dismiss it.

**Match:** This is the same problem class as any dismissible overlay/modal lacking a keyboard escape hatch — a standard, well-known UI pattern (e.g. `react-aria`, native `<dialog>` behavior) is to close on `Escape` in addition to backdrop clicks.

**Plan:**
1. Import `useEffect` alongside the existing `useState` import.
2. Add a `useEffect` scoped to `mobileMenuOpen`: while true, attach a `document`-level `keydown` listener that checks for `event.key === "Escape"` and calls the existing `setMobileMenuOpen(false)`.
3. Return a cleanup function removing the listener, so it doesn't leak when the sidebar closes or the component unmounts.
4. Confirm no other files are touched (`git status`).
5. Run `npx tsc --noEmit`, `npm run lint`, `npm run build` to catch any regressions.

**Review:** Confirm the fix matches the existing code style (same `setMobileMenuOpen(false)` call already used by the backdrop), doesn't introduce new dependencies, and the listener is properly scoped so it's never active when the sidebar is closed.

**Evaluate:** Root cause is the *absence* of any keyboard handler — not a bug in existing logic, but a missing feature. The fix is minimal and additive: no existing behavior (backdrop click, desktop layout, other key handlers) is touched or put at risk.

---

## Phase III — Build

### What I changed

**File:** `src/components/layout/DashboardLayout.tsx`
**Commit branch:** `fix/sidebar-escape-key`

Only one file was modified. Two changes were made:

**1. Added `useEffect` to the React import:**

```tsx
// Before
import React, { useState } from "react";

// After
import React, { useState, useEffect } from "react";
```

**2. Added a keyboard event listener after the existing `useState` declarations:**

```tsx
useEffect(() => {
  if (!mobileMenuOpen) return;

  const handleKeyDown = (event: KeyboardEvent) => {
    if (event.key === "Escape") {
      setMobileMenuOpen(false);
    }
  };

  document.addEventListener("keydown", handleKeyDown);
  return () => {
    document.removeEventListener("keydown", handleKeyDown);
  };
}, [mobileMenuOpen]);
```

### Why this approach

- **Scoped to open state:** The `if (!mobileMenuOpen) return` guard means the listener is only active while the sidebar is actually open. No unnecessary event listeners fire when the sidebar is closed.
- **Cleanup on close:** The returned cleanup function removes the listener every time `mobileMenuOpen` goes from `true` to `false`, preventing memory leaks.
- **No new dependencies:** The fix uses only React's built-in `useEffect` hook and the native browser `KeyboardEvent` API — nothing extra to install or maintain.
- **Consistent with existing code:** The fix calls the same `setMobileMenuOpen(false)` already used by the backdrop click handler, so the dismiss behavior is identical regardless of how the sidebar is closed.

### Files changed

| File | Change |
|---|---|
| `src/components/layout/DashboardLayout.tsx` | Added `useEffect` import + keyboard listener |

No other files were modified.

---

## Phase IV — Submit and Iterate

**PR Link:** *(add after opening)*

- [ ] Pull request opened
- [ ] CI checks green
- [ ] Maintainer feedback received
- [ ] Revisions pushed (if needed)
- [ ] PR merged

---

## Links

- [Issue #530](https://github.com/nisshchayarathi/gitverse-nextjs/issues/530)
- [My Fork](https://github.com/salabili212/gitverse-nextjs)
- [Feature Branch](https://github.com/salabili212/gitverse-nextjs/tree/fix/sidebar-escape-key)
- [Pull Request](#) *(update after opening)*
