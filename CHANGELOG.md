# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

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
