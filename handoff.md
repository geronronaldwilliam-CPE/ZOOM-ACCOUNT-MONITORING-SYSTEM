# Zoom Meeting Scheduler — Session Handoff

> **Instructions:** This file is updated at the end of every session to track what was built, what decisions were made, and what remains to do.

---

## Project Overview

A single-file web app (`index.html` + `script.js` + `styles.css`) for scheduling Zoom meetings across three capacity tiers (100/300/500 pax). The app has two views:
- **Public scheduler** — users browse a calendar and submit booking requests
- **Admin panel** — accessed via `#admin` hash, password-protected

Data is stored entirely in `localStorage` (no backend).

---

## Session 1 — 2026-07-07

### What was done

**Feature: Zoom Account Management in Admin Dashboard**

Added a full account management system so admins can define, manage, and track Zoom licenses per capacity tier (100/300/500 pax). The admin panel was also restructured with a tabbed navigation.

#### Changes by file

**`index.html`**
- Replaced the monolithic settings card in admin with a **tabbed admin navigation**: `Booking Requests` | `Zoom Accounts` | `Settings`
- Added the **Zoom Accounts panel** (`#apanel-accounts`) with a per-tier account grid rendered dynamically
- Added the **Add/Edit Account modal** (`#account-ov`) with fields: Name/Department, Capacity, License Status, Zoom Email, Expiry Date, Notes
- Added the **Confirm Delete modal** (`#delete-account-ov`)

**`script.js`**
- Added `loadAccounts()` / `saveAccounts()` / `defaultAccounts()` — localStorage-backed data layer for Zoom accounts
- `defaultAccounts()` seeds 3 accounts (one per tier: Main Conference Room 100pax, Auditorium A 300pax, Grand Hall 500pax)
- Added `genAccountId()` — auto-increments ACC-001, ACC-002…
- Added `renderAccountsGrid()` — groups accounts by capacity tier, renders each tier as a collapsible block with active/expired counts and per-account action buttons (Expire / Activate, Edit, Delete)
- Added `openAddAccountModal(presetCapacity)` — opens the modal pre-filled for a given tier
- Added `openEditAccountModal(id)` — populates modal with existing account data
- Added `saveAccount()` — creates or updates an account
- Added `openDeleteAccountModal(id)` / `confirmDeleteAccount()` — two-step delete confirmation
- Added `setAccountStatus(id, status)` — one-click Expire or Activate
- Added `rebuildUserRoomButtons()` — dynamically rebuilds the user-side room buttons and `f-pax` select from **active** accounts only; also updates the export pax dropdown
- Added `switchAdminTab(tab, btn)` — switches between Booking Requests / Zoom Accounts / Settings panels
- `showAdminPanel()` updated to load and render accounts on login
- `INIT` block now calls `rebuildUserRoomButtons()` on page load

**`styles.css`**
- Added admin tab bar styles (`.admin-tabs`, `.admin-tab`, `.admin-tab.active`)
- Added full Zoom Accounts section styles: tier block, tier header, account rows, status dots, status pills, expiry warning badge, account actions
- Retained all existing styles untouched

#### How the feature works (summary)

1. Admin goes to `#admin` → Zoom Accounts tab
2. Each capacity tier (100/300/500 pax) shows as a card with its accounts listed
3. Per account, admin can:
   - **Expire** an active license (turns it grey, removes from user-facing room buttons)
   - **Activate** an expired license (restores it)
   - **Edit** — name, capacity, email, expiry date, notes
   - **Delete** — with confirmation modal
4. Admin can add new accounts for any tier (e.g., a second 100 pax account named "MISS Department")
5. The user-side room buttons dynamically reflect only **active** accounts per capacity tier

---

## Session 2 — 2026-07-07

### What was done

**Feature: Dynamic User Dashboard Room Buttons (New Capacities + Expired Removal)**

#### Problem
When an admin added a Zoom account with a **new capacity** (e.g., 200 pax) that wasn't in the original 100/300/500 set, it did not appear on the user-facing dashboard. Likewise, there was no reliable mechanism to pick distinct colors for non-standard tiers.

#### Changes by file

**`script.js`**
- `rebuildUserRoomButtons()` rewritten with a **color cycle** (`ic-b → ic-t → ic-p → ic-g → ic-y`) so any new capacity tier automatically gets a distinct color on the user dashboard
- Added **auto-close calendar** if `activePax` tier is expired/deleted mid-session (calls `closeCal()` when the active pax is no longer in the active accounts list)
- Restored `f-pax` dropdown selection after rebuild to avoid resetting the user's choice
- Added `refreshCapacityOptions(selectVal)` — populates the Add/Edit Account modal's capacity `<select>` dynamically from all known capacities + a **"Custom (enter below)…"** option
- Added `toggleCustomCapInput()` — shows/hides a freeform number input when "Custom" is selected in the capacity dropdown
- `openAddAccountModal()` now calls `refreshCapacityOptions()` instead of hardcoding options
- `openEditAccountModal()` likewise uses `refreshCapacityOptions()` so the account's actual capacity is always selectable
- `saveAccount()` reads from the custom input when `acc-capacity.value === 'custom'`, with validation

**`index.html`**
- Added `onchange="toggleCustomCapInput()"` to `#acc-capacity` select
- Added "Custom (enter below)…" as a fallback option in the static HTML (overridden dynamically at runtime)

**`styles.css`**
- Added `.ic-g` (orange) and `.ic-y` (pink) color classes for the 4th and 5th capacity tiers

#### How it works

1. Admin adds a new Zoom account with capacity **200 pax** (via "Custom" option)
2. `saveAccount()` stores the account; `renderAccountsGrid()` → `rebuildUserRoomButtons()` fires
3. The user dashboard now shows a **200 pax** button with an auto-assigned color
4. If the admin **expires** that account (or deletes it), and no other 200 pax accounts remain active, the 200 pax button disappears from the dashboard immediately
5. If a user had the 200 pax calendar open when it expired, `closeCal()` is called automatically

---

## Known Limitations / Future Work

- [ ] No backend — all data in localStorage; refreshing on a different browser/device loses data
- [ ] Zoom account credentials (host key, API key) are not stored — this is purely a label/license tracker
- [ ] Export does not include account-level data (which specific Zoom account was used per booking)
- [ ] No drag-to-reorder accounts within a tier
- [ ] No auto-expiry detection on load (would need a `Date.now()` check against `acc-expiry`)
- [ ] Color cycle limited to 5 distinct colors; 6th+ tier will reuse colors

---

## File Inventory

| File | Purpose |
|---|---|
| `index.html` | Full app markup — user scheduler + admin panel |
| `script.js` | All logic — bookings, accounts, routing, calendar |
| `styles.css` | All styles — dark/light theme, user + admin |
| `handoff.md` | This file — session progress tracker |
