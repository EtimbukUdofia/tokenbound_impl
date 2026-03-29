# Pull Request

## Description

This PR delivers the **ticket purchase / browse events** experience in `soroban-client`, wires purchases to the **EventManager** contract via **simulate → assemble → Freighter (wallet) sign → Soroban RPC submit**, and **repairs merge-corrupted** client files that were breaking TypeScript and builds.

**Motivation:** Users need a real UI to list events, see details (dates, price, inventory), and buy tickets with clear success/failure and on-chain confirmation.

**Context:** `lib/soroban.ts`, `WalletContext`, `Header`, and `create-event` were in a broken state; dashboard and tests were updated to match the restored Soroban helpers.

Fixes #(issue)

## Type of Change

Delete options that are not relevant.

- [x] Bug fix (non-breaking change which fixes an issue)
- [x] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update
- [x] Code refactoring
- [ ] Performance improvement
- [x] Test coverage improvement (minor: test fixes for wallet connect / Header mocks)

## Changes Made

- **Events browse & purchase:** `EventCatalog` loads events from `getAllEvents`, adds search + filters (All / Upcoming / Canceled), card grid + detail modal, quantity selection, purchase via `purchaseTickets` (single ticket uses `purchase_ticket`; multiple uses `purchase_tickets`), success/error UI with tx hash, ledger, and Stellar Expert link (`getTxExplorerUrl`).
- **Routing:** `/events` remains primary; **`/browse` redirects to `/events`**; events page hero copy updated.
- **`lib/soroban.ts`:** Reimplemented contract helpers using `@stellar/stellar-sdk` `rpc` (simulate, `assembleTransaction`, `sendTransaction`, poll `getTransaction`); added **`purchaseTicket` / `purchaseTickets`**; **`createEvent` / `cancelEvent` / `updateEvent` / `claimFunds`** now take an explicit **`SignTransactionFn`**; reads use a configurable simulation source (wallet address or `NEXT_PUBLIC_SOROBAN_SIM_SOURCE`).
- **Wallet & layout:** Restored **`WalletContext`** and **`Header`** to consistent, compiling implementations.
- **Create event:** Restored **`create-event/page.tsx`** to a single working form using `createEvent(..., signTransaction)`.
- **Dashboard:** Passes **`signTransaction`** into mutating calls; **`getAllEvents(address)`** and **`getEventAttendees(eventId, address)`** aligned with new read APIs.
- **Tests:** Wallet test connect button fixed; Header mocks include **`signTransaction`**.

## Testing

Describe the tests that you ran to verify your changes:

- [x] `npx tsc --noEmit` (in `soroban-client`)
- [x] `npm run build` (Next.js production build)
- [x] `npx jest --no-watchman`

## Checklist

- [x] My code follows the style guidelines of this project
- [x] I have performed a self-review of my code
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings (ESLint may still flag pre-existing issues in `dashboard/page.tsx`, e.g. `Date.now` / `any`)
- [ ] I have added tests that prove my fix is effective or that my feature works
- [x] New and existing unit tests pass locally with my changes
- [ ] Any dependent changes have been merged and published

## Screenshots (if applicable)

Add screenshots to help explain your changes (e.g. `/events` listing, filters, detail modal, success state with tx link).

## Additional Notes

**Environment variables**

- **`NEXT_PUBLIC_EVENT_MANAGER_CONTRACT`** — required for reads and purchases.
- **`NEXT_PUBLIC_SOROBAN_SIM_SOURCE`** — optional funded **G**-address to simulate `get_all_events` / `get_event_buyers` when no wallet is connected.

**API note:** `buyTickets` was removed; use **`purchaseTickets(params, signTransaction)`**. Dashboard and create-event flows use the updated `createEvent` / `cancelEvent` / `updateEvent` / `claimFunds` signatures with an explicit signer.
