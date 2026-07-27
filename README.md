## Phase II — Reproduce and Plan

### Environment Setup
Hit a `Server Error: Missing required environment variables: INTERNAL_WORKER_SECRET`
on first run. Inspected `lib/env.ts` and found the app validates `DATABASE_URL`,
`NEXTAUTH_URL`, `GEMINI_API_KEY`, and `INTERNAL_WORKER_SECRET` on boot, and
requires either `TOKEN_ENCRYPTION_KEY` or `KMS_KEY_ID`.

**Challenge:** No local database or auth backend was available, and standing
one up was out of scope for a small UI fix.

**Resolution:** The project has a built-in validation skip for CI environments.
Used that flag to boot the dev server without a live database:
```powershell
$env:CI="true"; npm run dev
```
This got the app running on `localhost:3001`. Full authenticated pages (like
`/Dashboard`) still require a real database connection to render — confirmed
this when a signup attempt returned a backend error instead of a valid
response. Reproduction of the actual bug was therefore done at the code level
(see below), since the component and its bug are fully visible in the source
without needing a live session.

**Branch:** `fix/sidebar-escape-key` (pushed to my fork). Note: this doesn't
follow the `fix-issue-NNN` numbering convention exactly — for future issues
I'll use `fix-issue-530`-style naming from the start.

### Reproduction Steps
1. Clone the fork and run `npm install`.
2. Set `CI=true` and run `npm run dev` to bypass env validation.
3. Open `src/components/layout/DashboardLayout.tsx` and locate the mobile
   sidebar block (originally lines 117–156).
4. Search the file for any keyboard event handling:
```powershell
   Select-String -Path src\components\layout\DashboardLayout.tsx -Pattern "keydown|Escape|onKeyDown"
```
5. Observe the result: zero matches. The only dismiss handler present is
   `onClick={() => setMobileMenuOpen(false)}` on the backdrop `<div>`.

**Expected behavior:** Pressing Escape while the mobile sidebar is open
should close it, consistent with standard behavior for dismissible
overlays/dialogs.

**Actual behavior:** Pressing Escape does nothing. The sidebar only closes
via a mouse click on the backdrop overlay; there is no keyboard-triggered
close path at all.

### Plan (UMPIRE)

**Understand:** The mobile sidebar (`DashboardLayout.tsx`) opens as an
overlay controlled by `mobileMenuOpen` state. It currently closes only
through a single `onClick` handler on the backdrop `<div>`. There is no
keyboard accessibility path to dismiss it.

**Match:** This is the same problem class as any dismissible overlay/modal
lacking a keyboard escape hatch — a standard, well-known UI pattern (e.g.
`react-aria`, native `<dialog>` behavior) is to close on `Escape` in addition
to backdrop clicks.

**Plan:**
1. Import `useEffect` alongside the existing `useState` import.
2. Add a `useEffect` scoped to `mobileMenuOpen`: while true, attach a
   `document`-level `keydown` listener that checks for `event.key === "Escape"`
   and calls the existing `setMobileMenuOpen(false)`.
3. Return a cleanup function removing the listener, so it doesn't leak when
   the sidebar closes or the component unmounts.
4. Confirm no other files are touched (`git status`).
5. Run `npx tsc --noEmit`, `npm run lint`, `npm run build` to catch any
   regressions.

**Review:** Confirm the fix matches the existing code style (same
`setMobileMenuOpen(false)` call already used by the backdrop), doesn't
introduce new dependencies, and the listener is properly scoped so it's
never active when the sidebar is closed.

**Evaluate:** Root cause is the *absence* of any keyboard handler — not a
bug in existing logic, but a missing feature. The fix is minimal and
additive: no existing behavior (backdrop click, desktop layout, other
key handlers) is touched or put at risk.
