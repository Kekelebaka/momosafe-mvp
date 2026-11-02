# MoMoSafe sandbox MVP

Buyer-authorised handover workflow on Azure Functions, Cosmos DB and the MTN MoMo sandbox. No real money, currency conversion, insurance or licensing claim. Independent project, not an MTN-operated product.

Hosted sandbox: https://momosafe-053-sandbox.azurewebsites.net/

Current v5 source: [3-MoMoSafe-Azure-sandbox-source-v5.zip](3-MoMoSafe-Azure-sandbox-source-v5.zip). Extract for runnable source and SANDBOX-STATUS.md. Earlier archives are retained as history. The hosted landing page and anonymous HTTP boundaries are verified. Genuine hosted Google sign-in and full payment workflow testing are still in progress.

## One-login guided demo

An allowlisted Google owner can select Guided demo, then switch between seller, buyer and two demo operator seats. Records are isolated from pilot records. The banner and receipts state that one person plays all seats. This never counts as independent two-person dispute approval. Pilot mode still requires distinct real Google subjects for the two approvals.

## Workflow

Seller creates a ZAR or GHS product record. Buyer accepts, starting an MTN EUR sandbox collection. Authoritative status reads lock test funds. Seller creates a random one-use handover QR/token; buyer confirms after inspection to reserve the test seller payout. A 72-hour silence timeout reserves a test refund. A seller handover claim or participant dispute freezes the timeout for operator review. Two distinct matching operator approvals reserve release or refund.

Product currency is display/accounting denomination only. The numeric amount is sent as EUR canned test data, not converted to a real rand or cedi balance. Use fictional items and test data only.

## Build and tests

Node22+: `npm ci`, `npm test`, `npm run build`. 57 local tests cover the state machine, signed-session checks, sandbox adapter and HTTP boundaries. Deployment uses the resulting `dist/`, `azure-dist/`, production dependencies, `host.json` and `package.json`.

Cloud configuration: `MOMO_MODE=sandbox`, `APP_ORIGIN`, `GOOGLE_CLIENT_ID`, `SESSION_SECRET` (at least32 characters), `COSMOS_CONNECTION`, `AzureWebJobsStorage`, product-specific `MTN_COLLECTION_*` and `MTN_DISBURSEMENT_*` (`SUBSCRIPTION_KEY`, `API_USER`, `API_KEY`), `MTN_TEST_PAYER`, `MTN_TEST_PAYEE`. Operator emails use `ADMIN_EMAILS`; guided demo requires `GUIDED_DEMO_ENABLED=true` and `DEMO_OWNER_EMAILS`. Keep every secret in private deployment settings, never source.

## Structure

- `engine.ts`, `model.ts`, `store.ts`: state machine and storage contract.
- `sandbox-rail.ts`: MTN sandbox-only adapter, stable references, status reads.
- `azure.ts`, `cloud-http.ts`, `cosmos.ts`, `auth.ts`: Durable reconciliation, HTTP boundaries, transactional ledger and verified Google login.
- `cloud-web.tsx`: default hosted frontend.
- `*.test.ts`: unit and boundary tests.
- `LEGACY-MOCK.md`, `server.ts`, `web.tsx`, `preview.ts`: earlier local/device simulation. `npm run dev` starts that legacy local server, not the Azure backend. Build the device-only preview with `VITE_PREVIEW_MODE=device npm run build`.

## Not ready for real money

No production custody or MTN production onboarding. Runtime concurrency/restart, Durable replay/timers, backup restore, abuse/rate limits, retention/deletion and real independent operator tests remain unfinished. Camera scanning is not implemented; QR has copy/paste token entry. A50-deal sandbox guard is not complete rate limiting. Do not market this as a production payment service.


# MoMoSafe sandbox build status

2 November 2026. In progress, not pilot-ready.

## Two separate modes

- Existing hosted device preview remains a browser-local simulation, not shared state or real login: https://momosafe-mock-preview.pages.dev/
- Default source build now targets Azure Functions with Google sign-in, Cosmos transactional state and MTN sandbox HTTP adapters. Azure deployment is live at https://momosafe-053-sandbox.azurewebsites.net/ . Full signed-in cloud testing is unfinished.

## Implemented locally

Buyer/seller flows and ZAR/GHS product records; opaque one-use QR handover token; 72-hour silence refund; seller handover claim freezes timeout for review; disputes require two distinct matching operator approvals. No automatic seller payout on silence.

Azure HTTP routes, server-verified Google ID tokens and secure HttpOnly one-hour session cookie, origin checks, actual JSON body size limit, server-side operator allowlist, audited user restriction/ban storage, Cosmos ETag transactional state and ledger, Durable timers and reconciliation. Cloud mock identities and settlement simulation are absent. Pilot operator allowlist is empty. Owner-only guided demo permits one signed-in person to role-play separate buyer/seller/operator seats on isolated demo records. It is explicitly not independent two-person approval.

MTN uses https://sandbox.momodeveloper.mtn.com only, separate product credentials and tokens, stable persisted UUID references and status reads. All rail calls use EUR canned test data. Product ZAR/GHS amounts are not converted or held as real balances.

## Verified

- 45 local unit/boundary tests pass: 21 engine, 5 auth, 6 rail, 13 HTTP boundary. Auth network verification is not a real sign-in test.
- Root and Azure TypeScript compilation pass; cloud frontend production build passes.
- Actual hosted logged-out cloud UI inspected at 390px mobile and desktop. Signed-in screens need cloud inspection.
- Actual MTN success-case Collection and Disbursement: accepted202 then GET200 SUCCESSFUL, EUR1000, test MSISDN46733123465. Refs bdf922a9-0626-45b5-a961-3049df8a3823 and 5799cde3-6c18-4f24-a83a-5c024d871886. This is separate rail verification, not the hosted app's end-to-end flow.
- Azure ARM validation succeeded for Flex Consumption and single-region Cosmos serverless in South Africa North. Resources created in the approved team subscription: Flex FC1 max1/512MB/no always-ready, storage Standard_LRS, single-region Cosmos Serverless, database and container. API health200 confirms sandbox configuration, unauthenticated deal/session reads401. Google web client and owner test user configured. Owner reported successful genuine Google sign-in; separate verification session is waiting for phone MFA.

## Not yet verified or complete

Independent signed-in UI retest, runtime restart, Durable replay/timers, two-person end-to-end handover/release/refund, actual operator identities and two-person ruling, backup/restore, alerts, robust abuse/rate limits, pagination, retention and deletion workflow, callback host, camera scanning, PWA installation. A 50-deal guard is not complete rate limiting. No production or real money support.

## Cost and security boundaries

USD5 total Azure sandbox ceiling approved. No always-ready servers, paid SMS, paid hosting upgrade or custom domain. Consumption billing is not a hard cap; check usage and stop before the ceiling. No secrets in source. Private local deployment settings are excluded from archives and must be removed after configuration verification. Consumption cost follow-up is owned by the coordinator. No roles, policies or budgets are changed.

## Building

`npm ci`, `npm test`, `npm run build`. Default web build is the cloud UI. For the legacy device preview, `VITE_PREVIEW_MODE=device npm run build`. Local `npm run dev` still runs the legacy mock server and is not the Google/Cosmos backend.

## Create response correction

A first hosted user attempt saved DRAFT records but returned500 because Durable SDK getStatus throws on missing orchestration. Handler changed to list-and-find before starting; SDK-shaped regression passes. Stable frontend submit key prevents duplicate create after an ambiguous response. Corrected v4 deployed; signed-in retest pending. Two pre-fix DRAFTs are preserved, no collection started.

## Additional live checks, 18:48 SAST

- Engine + real Cosmos + MTN sandbox integration: collection SUCCESSFUL, LOCKED, one-use handover, release SUCCESSFUL, RELEASED, liability zero. Fresh Cosmos client read persisted RELEASED. Isolated EUR test record, not Google/UI end-to-end proof.
- Hosted API synthetic signed-session regression: create201, same-key retry201/same ID, authenticated list200. This deliberately does not verify Google sign-in.
- Cross-client Cosmos write race on the isolated released integration record: one commit, one conflict, state version increased exactly once and only the winning event persisted.

## v5 buyer invitation, 19:03 SAST

Approved explicit seller-click invitations are enabled for new pilot drafts, never Guided demo. The seller sees the buyer recipient before sending. One persisted invite job per deal, Cosmos ETag send lease, stable Resend deduplication key and23-hour retry cutoff prevent blind duplicate retry. Resend acceptance is not inbox confirmation. Existing earlier drafts have no invite job.

57 local tests pass. Hosted v5 create201, same-key retry201/same ID, authenticated list200, seller-only invite metadata200/PREPARED verified with synthetic test identity, no email sent in that check. This is not a Google sign-in test. First approved exact-template test sent only to the owner Gmail; Resend reports Delivered. Uses existing verified root domain and send-only key; no DNS/account/paid-plan change. Free plan quotas are shared with other apps. Stronger abuse limits, provider event handling and actual signed-in invite click testing remain unfinished. The invite screen was inspected using a local synthetic visual fixture, not genuine authentication.

Durable runtime activity and5-minute timer continue-as-new were observed in Azure. Real Cosmos+MTN sandbox isolated claim/dispute/refund passed, same approver blocked, second demo approval refunded and liability zero. This is not independent human operator proof. Deployment private settings/key copies removed after config verification; live app settings retained.
