# MoMoSafe MVP: local mock build

A working React PWA and TypeScript backend for demonstrating buyer-authorised escrow release. **No real money, MTN connection, Azure resources, OTPs or SMS.** Do not enter real personal information.

No affiliation with MTN is claimed. This is an independent mock demo, not an approved MTN product or a live payment service.

## Run

Node 22 or later:

```sh
npm ci
npm run dev
```

Open http://127.0.0.1:5173. The API binds to loopback at port 7071. Choose demo seller, create a deal, switch to buyer, accept, open simulation controls, simulate collection success. Switch to seller to show a one-time handover token; switch back to buyer, paste it and authorise release. Simulate payout success to produce the receipt. Tokens are deliberately visible in this mock UI.

`npm run check` runs domain tests, TypeScript checks, the web production build and Azure scaffold compilation. `npm run test:e2e` runs Chromium browser tests. Set `CHROME_PATH` if Chrome is not at `/usr/bin/google-chrome`.

## Implemented

- Seven business states from the Azure design: DRAFT, AWAITING_PAYMENT, LOCKED, DISPUTED, RELEASED, REFUNDED, EXPIRED. Payment operation status is separate.
- Integer cents, up to 1,000.00 in ZAR or GHS. Demo fee zero. Production pricing is undecided.
- Buyer/seller demo identities and two distinct demo admin roles. This is **not authentication**.
- Collection, release and refund simulated by a separate rail; state transitions require its status read, never a supplied callback result.
- Stable UUID payment references persisted before submission; one release OR refund reservation; terminal states immutable.
- 72-hour handover clock refunds on silence; dispute freezes automatic payout. Two distinct matching demo approvals resolve disputes.
- Random one-use handover tokens, server-side hashed, valid ten minutes; refresh invalidates previous token; buyer session required. QR rendering plus copy/paste token entry. Camera scanning is not implemented.
- Append-only logical event log, chronological timeline, receipts and liability balance.
- Atomic local JSON snapshot persistence in `.local/`, single-process command ordering.
- Offline shell and network-only API; web app manifest and app icon. Full browser installability/offline lifecycle not verified yet.
- TypeScript Azure Durable Functions v4 scaffold with activities, deterministic durable timers and reconciler. Cosmos transactional adapter.

## Intentionally different from the PDF sketches

1. The older design's seven states and integer cents are the reference. The later guide's extra operation states are stored separately.
2. A still-pending collection at 15 minutes is flagged overdue and remains under status reconciliation. Blindly marking it expired could orphan funds if the payment later succeeds. A production adapter must distinguish definitive failure from ambiguous timeout and define a late-success refund policy.
3. Cosmos uses **one container**, partition key `/escrowId`, with state and ledger documents in the same partition. An ETag-protected transactional batch commits state and new events together. Two containers cannot do this atomically. This adapter is compiled, not emulator/cloud tested.
4. A token is random and opaque rather than HMAC payload. Only its hash is stored. It is bound to the deal and buyer checks on the server. Possession is not proof of physical proximity.

## What is not finished

The **runnable application is local mock-only**. Azure code is a scaffold, not a deployed backend. The local Express routes are not yet wired to Azure HTTP triggers or Durable clients. Cosmos has not been exercised against an emulator or Azure. The cloud mock rail is process-local and cannot be used for a scaled deployment. Do not deploy this as a usable payment product.

Before Azure sandbox preview: confirm subscription and spending limits; choose region and Durable backend; provision storage/Cosmos/Functions/Static Web Apps; wire the command routes to durable instances; replace mock identities with real authentication and RBAC; define two real approvers; secure configuration and CORS; test Cosmos ETag contention, transaction limits and durable replay/restart behavior; add actual sandbox Collection/Disbursement adapters and encrypted durable operation persistence; verify callback host, late collection policy, redelivery and reconciliation; exercise real MTN sandbox test cases; add rate limits, alerts, pagination and retention. The local 3-second web poll is only local API traffic.

Before real money: licensed custody/safeguarding partner and legal review, MTN production onboarding, actual authorisation controls, operational monitoring, payout ambiguity/manual review, production data privacy and security review. No statement of licensing or insurance is made.

## Structure

`web.tsx`, `style.css`, `public/`: UI and PWA shell.
`server.ts`: loopback-only local HTTP adapter.
`model.ts`, `engine.ts`: validation and state machine.
`store.ts`: memory and JSON stores.
`momo.ts`: in-process simulator, intentionally no HTTP client.
`azure.ts`, `cosmos.ts`, `host.json`: compile-tested Azure scaffold.
`engine.test.ts`, `ui.spec.ts`: domain and browser tests.
`package-lock.json`: exact dependency lock.

## Test evidence (2 November 2026)

16 domain tests and 4 browser tests passed locally. TypeScript web and Azure code compiled. Web JS is about 105 KB gzip. Mobile 390px and desktop layouts inspected, including QR and final receipt. Not tested: deployed Azure, actual MTN sandbox, Cosmos runtime, durable runtime/replay, camera scan, browser PWA installation. No paid service was activated.

Privacy: local data remains in `.local/escrows.json`; demo sessions live in memory and expire on process restart. Delete `.local/` to reset. Do not expose the server beyond loopback. The demo allows identity switching and simulated settlement by any demo participant by design.

Dependency review: full npm audit (runtime and development) reported zero vulnerabilities after patch updates. uuid is overridden to 11.1.1 for the Durable dependency; cloud SDK behavior still needs runtime validation.

## Hosted device-only preview

https://momosafe-mock-preview.pages.dev/

Build with `VITE_PREVIEW_MODE=device npm run build`. The preview adapter (`preview.ts`) runs the exact same TypeScript escrow engine in-browser, not on Azure or Cloudflare Functions. State stays in browser localStorage; identity in sessionStorage. Tabs/devices are not a shared backend, and browser-local state is user-editable. The 72-hour timer is evaluated while the app is open, or on reopening, not a server guarantee. It is a click-through working simulation, not a payment service. ZAR default and GHS picker; no conversion. MTN's real sandbox currency constraints are not changed by these mock labels. Camera scan is not implemented; use the copy/paste token.

Live checks: full seller/buyer create-collect-handover-release-receipt sequence, 360px and 390px mobile, desktop, keyboard focus. No Azure, real MTN, real users or money were tested. This source archive contains both local server and device-only preview modes.
