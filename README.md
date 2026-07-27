## Phase II — Reproduce and Plan

### Environment Setup
Hit a `Server Error: Missing required environment variables: INTERNAL_WORKER_SECRET`
on first run. The app validates `DATABASE_URL`, `NEXTAUTH_URL`, `GEMINI_API_KEY`,
and `INTERNAL_WORKER_SECRET` on boot (`lib/env.ts`). Bypassed validation for local
UI-only testing using the project's built-in `CI=true` skip flag:
```powershell
$env:CI="true"; npm run dev
```
This let the dev server boot without a live database. Attempting to sign up beyond
this point failed with a backend error, confirming the app needs a real database
connection to fully render authenticated pages. Standing up a full Postgres +
auth flow was out of scope for this fix, so reproduction was done at the code
level instead.

### Reproducing the Bug (Code-Level)
Reviewed `src/components/layout/DashboardLayout.tsx` (lines 117–156). Confirmed
the mobile sidebar's backdrop overlay only had `onClick={() => setMobileMenuOpen(false)}`
— no `onKeyDown`, `keydown`, or `Escape` handling existed anywhere in the file
prior to the fix (verified via `Select-String -Pattern "keydown|Escape|onKeyDown"`
— zero matches).

### Plan
1. Import `useEffect` alongside the existing `useState` import.
2. Add a `useEffect` scoped to `mobileMenuOpen` state: attach a `document`-level
   `keydown` listener only while the sidebar is open, check for `Escape`, and call
   the existing `setMobileMenuOpen(false)`.
3. Clean up the listener when the sidebar closes or the component unmounts.
4. Confirm only `DashboardLayout.tsx` is modified (`git status`).
5. Run `npx tsc --noEmit`, `npm run lint`, `npm run build` to confirm no new
   errors are introduced.
6. Push and open a pull request referencing #530.

## Phase III — Build

### Implementation
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

### Verification
- `git status` confirmed only `DashboardLayout.tsx` was modified.
- `npx tsc --noEmit`, `npm run lint`, and `npm run build` all showed only
  **pre-existing, unrelated** errors (`CodeDependencyGraph.tsx`,
  `analysisJobService.ts`) — none introduced by this change.
- `npm run format` was run but **not committed**, since it reformatted the
  entire repository (hundreds of unrelated files) rather than just the
  changed file — committing that would have violated the "no unrelated
  formatting changes" guideline.

## Phase IV — Submit and Iterate

### Pull Request
Opened: [nisshchayarathi/gitverse-nextjs#2656](https://github.com/nisshchayarathi/gitverse-nextjs/pull/2656)

Branch: `fix/sidebar-escape-key` (pushed to my fork, PR opened against
upstream `main`)

**Status:** Open, awaiting maintainer review.

Will update this section with maintainer feedback and any follow-up
revisions as the review progresses.
