# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## [1.0.9] - 2026-07-10

### Changed
- **Mobile layout** — Balance stat now spans the full width with a larger figure, with Change and Accounts splitting the row below (desktop keeps the 3-across row). Header action buttons collapse to icons on phones so all three fit at 390px. Slightly tighter page and chart-card padding gives the chart more width on phones, and the chart is taller (240px).
- **Chart style** — smoother monotone line interpolation (no overshoot dips), gradient fill sized to the live chart area, dashed crosshair under the touched point, themed tooltip with a formatted date title (e.g. "12 Mar 2025"), hover point with a card-coloured ring, much subtler year/quarter dividers and horizontal grid lines, and compact y-axis labels ($12.5k, cents shown only on tiny ranges) that keep the axis narrow and never duplicate.
- **Date range header** — now formatted ("1 Jan 2024 → 10 Jul 2026") instead of raw ISO dates.

### Fixed
- **Pinch to zoom** — now anchors at the fingers' midpoint instead of the view centre, and lifting both fingers after a pinch no longer registers as a double-tap that instantly reset the zoom. Double-tap reset now only fires on two genuine quick taps.
- **Panning** — added: one-finger horizontal drag pans the zoomed chart on touch (vertical swipes still scroll the page — the canvas uses `touch-action: pan-y` instead of `none`), mouse drag and horizontal trackpad swipe pan on desktop, and trackpad pinch / ctrl+wheel zooms anchored at the cursor. Double-click resets on desktop.
- **X-axis ticks when zoomed** — tick placement now looks only at the visible window, so a zoomed-in view always has labels; tight zooms switch to day-level labels ("12 Mar").
- **Accounts grid** — columns use `minmax(0, 1fr)` so a long balance can no longer stretch its card wider than half; both cards are always equal halves. Balances that still don't fit are ellipsised.

---

## [1.0.8] - 2026-07-10

### Added
- **Auto-refresh on open** — when the page loads with a complete cache, it now silently checks the Up API for transactions newer than the most recent cached one and merges them in. If the check fails (offline, expired token), the cached dashboard stays up and a console warning is logged instead of an error screen.
- **Refresh button** — new header button that runs the same incremental check on demand: fetches only transactions since the last cached `newestSettledAt` and merges them, showing a spinner on the button while it runs. Distinct from **Resync**, which still clears the cache and re-downloads everything.

### Fixed
- `refresh()` existed in the codebase but was unreachable from the UI, referenced the wrong button id, and could leak the progress-ticker interval on error. It is now wired to the Refresh button, guarded against concurrent runs, and cleans up the progress UI on failure.

---

## [1.0.7] - 2026-06-01

### Added
- **Resync action** — new header button that clears all cached transaction data and re-downloads everything from scratch while keeping the API token. Requires confirmation before proceeding.
- **Confirmation modals** — both Resync and Disconnect now present an animated bottom-sheet modal (slides up on mobile, centres on desktop) with a description of what the action will do and explicit Cancel / confirm buttons. Tapping outside the modal dismisses it.
- **Disconnect with confirmation** — the previous silent disconnect button has been replaced with a deliberate modal-gated action.
- **`doResync()` / `promptResync()` / `promptDisconnect()` / `showModal()` / `closeModal()` / `confirmModal()`** — new JS functions managing the modal system and resync flow.

### Changed
- **Complete visual rebrand to upzen** — removed all Up Bank orange. New design language uses CSS custom properties throughout (`--accent`, `--bg`, `--surface`, `--text`, `--text-muted`, etc.) with a single dark-mode block that overrides the variable values, eliminating all previous CSS cascade ordering issues.
- **Colour palette** — light mode: warm cream background (`#F0EDE8`), forest green accent (`#2D6A4F`). Dark mode: deep ink-green background (`#0D1410`), sage green accent (`#52B788`). Positive/negative states use green and red respectively in both modes.
- **Typography** — added Google Fonts: *DM Serif Display* (italic) for the brand wordmark and setup headline; *DM Sans* for all UI text. Numbers use `font-variant-numeric: tabular-nums` for stable column alignment.
- **Header wordmark** — "Up Wealth" replaced with the upzen logotype: *upzen* in italic DM Serif Display (accent colour) + "wealth" in DM Sans (muted).
- **Header actions** — "↻ Refresh" and text "Disconnect" replaced with icon + label buttons: a circular-arrows Resync button and a log-out-arrow Disconnect button, both styled with appropriate danger/primary treatment.
- **Setup screen** — redesigned with an upzen eyebrow label, an italic serif headline ("Your balance, over time."), and a cleaner card layout for the token input.
- **Stat cards** — use `var(--surface)` with a thin border; labels are smaller and subtler; values use tabular numeric figures.
- **Chart** — line and gradient fill updated to the new accent green. Grid lines and tick colours inherit from the new variable system.
- **PWA icon** — regenerated in forest green (`#2D6A4F`) to match the new palette.
- **`theme-color`** — updated to `#2D6A4F` (light) and `#0D1410` (dark) via dual `<meta name="theme-color">` tags.
- **App title** — changed from "Up Wealth" to "upzen wealth" in `<title>`, `apple-mobile-web-app-title`, and footer.
- **Footer** — simplified to "v1.0.7 · About · upzen.co".

---

## [1.0.6] - 2026-06-01

### Fixed
- **Dark mode chart background (CSS ordering bug)** — the `.chart-card` dark mode override (`background: #1c1c1a`) was declared before the base `.chart-card` rule (`background: #fff`) in the stylesheet. Because both selectors have equal specificity, the later rule won, leaving the chart on a white background in dark mode. The dark mode override has been moved to a media block that appears after the base rule, so cascade order is now correct. Grid lines and the fill gradient are now correctly visible against the dark background.
- **Crash-resume estimation accuracy** — when recovering from a mid-load crash, the estimated total and ETA were computed only from the newly-fetched older transactions, ignoring the transactions already saved to disk. The estimate is now pre-seeded from the full set of existing transactions before any new fetching begins, then updated from all transactions (existing + newly fetched) after each page. The `null` sentinel for `estimated` in `setProgress` preserves the current estimate when the caller does not have a better one.
- **Duplicate month labels on x-axis** — when the chart covered a short time range (e.g. during initial incremental loading), Chart.js could place multiple ticks within the same calendar month, producing labels like "Jan, Jan, Feb, Feb". The `afterBuildTicks` hook now tracks the first index seen per `YYYY-MM` key and ensures at most one tick per month regardless of data density or zoom level.

### Added
- **PWA home screen icon** — added `icon.png` (180 × 180, orange background with white waveform) referenced via `<link rel="apple-touch-icon">` and `<link rel="icon">`. Added `apple-mobile-web-app-capable`, `apple-mobile-web-app-status-bar-style`, `apple-mobile-web-app-title`, and `theme-color` meta tags for a native-feeling iOS PWA experience.
- **`computeEstimate(txs, startMs)` helper** — extracted from inline estimation logic in `fetchAll` and `fetchOlderThan` into a shared function. Computes observed transaction density over the date range present in `txs`, then extrapolates to the full account lifetime using `startMs` as the fixed denominator.

### Changed
- **Faster PWA startup** — IndexedDB is now opened eagerly at module load time via a module-level `_dbPromise`, rather than lazily on the first `_idbGet` call inside `init()`. This removes one round-trip from the critical path between page parse and first meaningful render.
- **Pure line graph** — removed all data point dots (`pointRadius: 0` unconditionally). A subtle hit radius (`pointHitRadius: 10`) is preserved so tooltips still trigger on hover/touch.
- **Smart x-axis tick fallback** — when the visible range contains no year boundaries (short history or tight zoom), the axis now falls back to month-level ticks (max one per month, thinned by a step factor if crowded) rather than leaving Chart.js to place arbitrary ticks that may duplicate months.

---

## [1.0.5] - 2026-06-01

### Added
- **Instant dashboard** — the dashboard is now shown immediately after accounts are fetched (typically under one second), rather than waiting for the full transaction history to load. Account balances and card names are visible from the start.
- **Live chart filling** — the chart renders incrementally as each page of 100 transactions arrives. The line grows leftward (oldest to newest) in real time, so users can watch their full balance history take shape rather than staring at a loading screen.
- **Progress bar on dashboard** — the transaction progress bar and ETA countdown are now displayed inline on the dashboard (between the stats and chart), rather than on a separate loading screen. The bar auto-hides 600ms after loading completes.
- **Crash resume shows existing data instantly** — when recovering from a crash, the dashboard now shows immediately with all previously saved transactions. The chart extends leftward as older transactions are fetched, consistent with the normal load experience.

### Changed
- **`renderDashboard` updates chart in-place** — if a chart instance already exists, `renderDashboard` now updates `labels` and `data` directly and calls `chartInst.update('none')` rather than destroying and recreating the `Chart` instance. This eliminates flicker and avoids re-initialising the gradient, quarter-grid plugin, and pinch-to-zoom touch handlers on every page.
- **Empty series handled gracefully** — `renderDashboard` now accepts an empty series (`[]`) without crashing. Balance is read from live account data, the change stat shows `—`, and the date range shows `Loading history…` until transactions arrive.
- **Loading screen simplified** — the loading screen is now shown only briefly while the accounts request resolves (usually < 1s). It no longer hosts the progress bar, which has moved to the dashboard.
- **Dark mode grid line contrast improved** — new year vertical lines bumped from `rgba(255,255,255,0.25)` to `0.45`; quarter dividers from `0.08` to `0.20`; y-axis horizontal grid lines from `0.06` to `0.15`. Tick label colour changed from `#777` to `#999` in dark mode.
- **Smart x-axis tick labels** — replaced `maxTicksLimit: 6` with a custom `afterBuildTicks` hook. Year boundaries (Jan 1) are always shown, labelled as the year number ("2024", "2025"). Quarter boundaries (Apr/Jul/Oct 1) are added only when available pixel width per tick exceeds 58 px, labelled as "Q2"/"Q3"/"Q4". When zoomed in to a range with no year or quarter boundaries, Chart.js default tick placement is used and labels fall back to abbreviated month names ("Jan", "Feb", etc.).

---

## [1.0.4] - 2026-05-31

### Added
- **Crash recovery** — if the app is closed or crashes mid-load, it automatically resumes from where it left off on next launch. The init sequence detects a `loadComplete: false` flag in IndexedDB, shows "Resuming — X transactions already saved", then fetches only the missing older history using `filter[until]=<oldest_createdAt>` and merges it with the already-saved data before marking the load complete.
- **Real-time transaction writes** — every 100-transaction page is flushed to IndexedDB immediately via `savePartial()` as it arrives, rather than waiting for the full fetch to complete. Accounts are also written to disk as soon as they are fetched, before any transactions are loaded.
- **Token saved on input** — the API token is persisted to `localStorage` on every keystroke via `oninput`, before the Connect button is pressed. Ensures the token survives a crash on the loading screen and is prefilled automatically on reload.
- **`fetchOlderThan(until)`** — new internal function that fetches transactions with `filter[until]=<ISO 8601 timestamp>`, used exclusively by the crash recovery path to retrieve only the history that was not saved before the crash.
- **`savePartial(accounts, txs)`** — new internal function that writes in-progress state to IndexedDB with `loadComplete: false`. Distinct from `saveCache()` which computes `newestSettledAt` and sets `loadComplete: true`.
- **`_getRawCache()`** — new internal function that reads the raw IndexedDB record without building the chart series, used by `init()` to branch between normal load, crash recovery, and first-run states without unnecessary computation.
- **`accountStartMs(accounts)` helper** — extracts the earliest account `createdAt` timestamp from the accounts array; shared between `fetchAndRender`, `refresh`, and `resumeLoad` to avoid duplicating the reduction logic.

### Changed
- **`loadComplete` flag** — IndexedDB cache records now carry `loadComplete: boolean`. `false` during any in-progress load; `true` only once all transactions are fetched and `saveCache()` has been called. `init()` branches on this flag to decide whether to show the dashboard, trigger crash recovery, or show the setup screen.
- **`fetchAll` signature** — added optional `onPage(page, accumulated)` callback, called after each page is appended. Used by `fetchAndRender` and `refresh` to flush partial data to IndexedDB incrementally.
- **Progress bar seeded from existing data on resume** — `_prog.fetched` is initialised to the number of already-saved transactions before resuming, so the progress percentage and ETA account for work already done rather than restarting from zero.

---

## [1.0.3] - 2026-05-31

### Fixed
- **Transaction estimate denominator** — the progress bar estimate was fundamentally broken: it used the oldest transaction *seen so far* as the age baseline, which caused the estimate to collapse to roughly the number of transactions already fetched rather than projecting the full history. The fix uses the account's actual `createdAt` date as a fixed denominator, so `txPerDay × accountAgeDays` correctly extrapolates across the full account lifetime and improves with each page loaded.
- **Cache not persisting across reloads** — switched from `localStorage` to `IndexedDB`, eliminating silent failures caused by the browser's 5 MB `localStorage` quota being exceeded for users with large transaction histories. The token remains in `localStorage` as a lightweight fallback.
- **Legacy cache migration** — existing data stored under the old `upwealth_v3` `localStorage` key is automatically migrated to IndexedDB on first load, then the legacy key is removed. No data loss, no re-authentication required.
- **All cache operations are now properly async** — `saveCache`, `loadCache`, `clearCache`, `reset`, and `init` are all `async/await`, eliminating potential race conditions where cache writes were fire-and-forget.

### Added
- **iOS PWA automatic update check** — when launched as a home screen app, the PWA silently fetches the current page with `cache: no-store` on every launch and compares the `APP_VERSION` string. If a newer version is detected, it forces a hard reload with a cache-busting URL, ensuring users always run the latest published version.
- **Cache-Control meta tags** — added `no-cache, no-store, must-revalidate` HTTP-equivalent meta tags as a belt-and-suspenders complement to the JS update check.

---

## [1.0.2] - 2026-05-31

### Fixed
- **Negative balance at chart origin** — reconstructed historical balances are now clamped to `Math.max(0, bal)`. Up does not support overdrafts, so negative values were always a reconstruction artefact from large early deposits.
- **Token not persisting across reloads on GitHub Pages** — the API token is now stored in a dedicated `localStorage` key (`upwealth_token`) independently of the main cache payload. Previously, if the cache write failed silently due to quota limits, the token was lost on reload.
- **Redundant series removed from cache** — the chart series is no longer stored in the cache payload; it is reconstructed from `accounts` + `txs` on load. This reduces the cache size and gives more headroom before hitting storage limits.

### Added
- **Transaction loading progress bar** — an animated progress bar appears while transactions are being fetched, switching from indeterminate to percentage-based once the first page arrives.
- **Rolling transaction estimate** — after each 100-transaction page, the estimated total is recalculated from the account's creation date and observed transaction density, making the percentage more accurate over time.
- **ETA countdown** — a live estimated time remaining is displayed alongside the progress bar, ticking down every second between page arrivals.
- **Chart pinch-to-zoom** — two-finger pinch to zoom into a time range; double-tap to reset to full view.
- **Quarter and new year grid lines** — the chart now draws subtle dashed vertical lines at quarter boundaries (Apr, Jul, Oct) and solid darker lines at each new year (Jan 1).
- **Footer** — version number, About link (GitHub README), and "Created by upzen.co" added to the bottom of the page.

### Changed
- **Text contrast improvements** — explicit colours set for stat values, account balances, stat labels, section labels, and account names in both light and dark mode, meeting a minimum contrast target across all elements.
- **Header title contrast** — explicit foreground colour set for the "Up Wealth" title in both light and dark mode, replacing inherited values that were too dim in dark mode.

---

## [1.0.1] - 2026-05-31

### Added
- **Privacy badge** — "Your token never leaves your device" badge with shield icon displayed on the setup screen beneath the token input.
- **Unofficial tool disclaimer** — added to the setup screen and README: "Unofficial tool — not affiliated with or endorsed by Up Banking."
- **README** — initial GitHub README covering privacy, getting started, how the chart works, caching behaviour, technical details, and disclaimer.

### Changed
- **Prevent browser pinch-to-zoom** — viewport meta updated with `user-scalable=no, maximum-scale=1.0` to prevent accidental page zoom on mobile.
- **Canvas touch-action** — set `touch-action: none` on the chart canvas so the browser defers all touch events to JavaScript.

---

## [1.0.0] - 2026-05-31

### Added
- Initial release.
- Single-file HTML app — no server, no backend, no build step.
- Connects to the Up Banking API using a Personal Access Token.
- Reconstructs historical balance from transaction history by walking backwards from the current account balance.
- Per-account balance breakdown for all personal (individual) accounts.
- Responsive chart with dark mode support via `prefers-color-scheme`.
- Smart caching with incremental refresh — only fetches transactions newer than the most recent cached one.
- Disconnect button clears all cached data and the token from the device.
