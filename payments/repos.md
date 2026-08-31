# Payment-domain repo catalog

Every payment-related repo, what it does, and where it sits in the AutoPay/recurring-billing
flow. Mined from the repos on disk 2026-08-31, cross-checked against
[autopay-hld.md](autopay-hld.md) (its §11 repo map is the code-anchored authority for AutoPay
roles). Local clones live under `C:\Users\Nikhil.Jain\Code\`.

| Repo | What it is | AutoPay role |
|---|---|---|
| `Mindbody.Payments.AutoPay` | Modern AutoPay orchestration service (.NET 10) | Core: run scheduling + execution (Pipelines A/C, B2 charge client) |
| `Mindbody.Web.Scripts` | Legacy nightly scripts (classic ASP + .NET Fx host) | Core: legacy charge path B1 — `adm_daily_autopay.asp`, `inc_run_eft.asp`, `inc_ccp.asp` |
| `Mindbody.Api.Payments` | Legacy Payments API v1 (.NET Fx 4.8 Web API) | Charge API — same `PaymentTransactionProcessingController` serves B1-direct and B2 |
| `Mindbody.Services.Marketplace` | Legacy payments/cart/checkout Windows Services (WCF + MassTransit) | Gateway dispatch host — MarketplacePayments executes charges |
| `Mindbody.Map.PaymentProcessingV2` | Legacy gateway-abstraction NuGet library (`Mindbody.Map.PaymentProcessing`) | Bottom of the money path — IGateway over Stripe/MBPS/legacy PSPs, lives inside Marketplace |
| `Mindbody.Payments.CardNotPresent` | Core CNP payments service, PCI Cat-1 (.NET 10) | Executes recurring card charges; PSP selection (Stripe/GP/Xendit) and MIT tagging happen here |
| `Mindbody.Payments.Bank` | Bank-debit payments service (ACH, SEPA, local rails; .NET 10) | Bank + wallet (Method 801 `/paymentintent`) recurring lanes |
| `Mindbody.Payments.CardPresent` | In-person terminal transactions, PCI Cat-1 (.NET 10) | Feeds AutoPay: terminal-captured cards vaulted for later recurring use |
| `Mindbody.Payments.Instruments` | Payment-instrument vault + NGP token indirection (.NET 10) | Detokenization + `IsAutoPay` instrument store; orphan-token sweep |
| `Mindbody.Pricing` | Pricing Domain API (.NET 8, GraphQL) | Fee chain: AutoPay API `autopay-fees/calculate` → Pricing `v2/payment-method/fees` (`FeeController.cs:15-56`) |
| `Mindbody.Sales` | Sales Domain API (.NET 10) | Observe-only: `AutoPayTotalEqualsContractAmountValidator` audits schedules vs contract amounts |
| `Mindbody.Payments.Sessions` | Brand-agnostic payment sessions/links orchestration (.NET 10) | Not in the AutoPay path (verified) — supports recurring line items for its own sessions |

Referenced by the HLD but **not cloned locally** (claims about them are inherited **[I]**):
Card Updater / `CardSynchronizer` · `Mindbody.Web.Clients` (autopay admin/domain helpers) ·
Tokenizer service · `GlobalNgpTokenLookup` SQL artifacts.

---

## Mindbody.Payments.AutoPay
Modern AutoPay service owning scheduled recurring billing: ASP.NET Core API (including the
`AspMigration` controllers that replace legacy nightly script calls) plus EventBridge/SQS-driven
Lambdas (`CreateScheduleRuns` → `ProcessScheduleRuns` → `ProcessStudioRuns` → `CompleteScheduleRuns`)
that create, gate, fan out, and close billing runs. Charges execute by calling the Payments API.
- Stack: .NET 10, ASP.NET Core + AWS Lambdas, MongoDB (run ledger), PostgreSQL, legacy SQL Server
  projects, SQS/SNS, LaunchDarkly, Helm/Tilt/Docker.
- Talks to: Payments API (`PaymentApiClient`), Subscriber API, Pricing API, LaunchDarkly, Asimov auth.
- Key docs: README.md (template boilerplate — the real design doc is [autopay-hld.md](autopay-hld.md)).

## Mindbody.Web.Scripts
Legacy "Nightly Scripts": classic ASP pages on a JAMS schedule (IIS/EC2) that settle transactions,
process returns, and run the daily AutoPay charge night — `adm_daily_autopay.asp` (+`_v2`),
`inc_run_eft.asp` (sale composition, status writes, renewal), `inc_ccp.asp` (charge calls, Gate 3).
- Stack: Classic ASP + .NET Framework 4.5.2 host (`ScriptsFramework`/WebNew), SQL Server, Redis, JAMS.
- AutoPay: the B1 lane and the shared `runEFT` charge engine both live here; being displaced by
  Mindbody.Payments.AutoPay but still executes every charge today.
- Key docs: README.md.

## Mindbody.Api.Payments
Legacy Payments API v1 (.NET Framework 4.8, Web API 2): Subscriber payment methods, cart,
card-present, voids/returns, and the transaction-processing routes. No README — inferred from code.
- AutoPay: `PaymentTransactionProcessingController` (`:64` RoutePrefix, `:179` CreditCard, `:738`
  Bank, `:783` PaymentMethodId) is the single charge API for both the legacy B1-direct calls and
  the modern B2 calls — "modern vs legacy" is the payload, not the service.
- Downstream: Marketplace Payments over WCF net.tcp.

## Mindbody.Services.Marketplace
Legacy payments/shopping-cart/checkout logic as Windows Services (Marketplace, MarketplacePayments,
two SNS/SQS event processors); source of the `Mindbody.Contract.Marketplace*` NuGet contracts.
- Stack: .NET Framework Windows Services, WCF + MassTransit, SNS/SQS, SQL Server, LaunchDarkly.
- AutoPay: hosts the WCF entry the Payments API calls; dispatches to CNP/Bank/CP via the
  `Mindbody.Map.PaymentProcessing` NuGet (`MerchantPaymentProcessor` → `MbpsGateway`).
- Key docs: README.md, docs/bnpl-discovery-fee-rollout.md.

## Mindbody.Map.PaymentProcessingV2
Legacy gateway-abstraction class library (packaged as the `Mindbody.Map.PaymentProcessing` NuGet —
a library inside Marketplace, **not** a service hop): common `IGateway` over Stripe, MBPS,
TransFirst, Elavon, Ezidebit, HSBC, Moneris, BlueFin, Paysafe, and more, for charging, refunds,
and tokenization.
- Stack: .NET Framework 4.8 class library, packages.config era, SOAP web references for old PSPs.

## Mindbody.Payments.CardNotPresent
The Payments Integration Gateway (PIG): core card-not-present payments service — e-commerce and
stored-card charges, refunds, tokenization facade, fees/surcharges, network-transaction records —
routing each transaction to the right PSP. PCI DSS Category 1.
- Stack: .NET 10, MongoDB, per-PSP gateway projects (Stripe, GlobalPayments, Xendit), SNS/SQS
  Lambda (`NetworkTxnProcessor`), LaunchDarkly, spec-kit.
- AutoPay: recurring card charges execute here; MIT/OffSession tagging and PSP selection are
  CNP-internal; NetworkTransactionIds for merchant-initiated charges are upserted here.
- Key docs: README.md, AGENTS.md, docs/pattern-inventory.md.

## Mindbody.Payments.Bank
Bank-debit payments service: payment intents and charges for ACH, SEPA, and local rails, with
explicit `IsRecurringPayment`/`SetupForRecurringPayment` support and async settlement processing
(`CheckoutRun` Lambda, Xendit SNS processor). README overview is TODO — inferred from code.
- Stack: .NET 10, MongoDB, AWS Lambdas, Stripe/GlobalPayments/Xendit, LaunchDarkly.
- AutoPay: the bank-rails recurring lane, and — per the HLD — the wallet lane too (Method 801
  Apple Pay rides Bank `/paymentintent`, the root of the wallet MIT gap).

## Mindbody.Payments.CardPresent
Centralizes card-present (terminal) transactions: terminal charges, refunds, webhooks via Stripe
Terminal and GlobalPayments, plus vaulting cards captured at the terminal. PCI DSS Category 1.
- Stack: .NET 10, Helm/Arcus, spec-kit constitution.
- AutoPay: not the recurring path, but terminal-captured cards become stored instruments that
  recurring billing later charges through CNP.
- Key docs: README.md, .specify/memory/constitution.md, docs/repo-pattern-inventory.md.

## Mindbody.Payments.Instruments
PCI-scoped payment-instrument vault: tokenized cards and NGP tokens in MongoDB, card entry,
search, detokenization (TokenEx tokenizer behind it), funding-type identification via Stripe,
card access tokens; Interac refunds module and cron worker.
- Stack: .NET 10, MongoDB + PostgreSQL, Stripe.net, TokenEx, LaunchDarkly.
- AutoPay: the instrument store — saved cards carry `IsAutoPay`; detokenization runs through the
  `paymentTokens` facade the charge path uses; hosts the orphan-token sweep.
- Key docs: CLAUDE.md, README.md, AppSettings_README.md.

## Mindbody.Pricing
Pricing Domain API: demand-based price adjustments (campaigns, rules) — API Modernization family.
- Stack: .NET 8, ASP.NET Core + GraphQL, no local DB (calls other domain APIs), LaunchDarkly.
- AutoPay: in the fee chain — ASP `CalculateAutopayFees` → AutoPay API
  `v1/subscribers/{id}/autopay-fees/calculate` → Pricing `v2/payment-method/fees`
  (`FeeController.cs:15-56`; HLD §2, code-anchored). The repo-level scan alone missed this —
  trust the HLD anchor.
- Key docs: README.md, docs/SETUP.md.

## Mindbody.Sales
Sales Domain API: purchase-transaction construction/execution, auditing, adjustments/refunds,
reciprocity settlement; includes the observe-only sale-integrity validation system.
- Stack: .NET 10, ASP.NET Core + Lambdas (SaleValidator, PaymentSessions, LegacyEnricher),
  Helm/helmfile, spec-kit constitution.
- AutoPay: observe-only — `AutoPayTotalEqualsContractAmountValidator` flags schedules that don't
  match contract amounts ($0.01 tolerance); reads `tblEFTSchedule`, never writes sales.
- Key docs: README.md, .speckit/constitution.md, docs/SETUP.md.

## Mindbody.Payments.Sessions
Brand-agnostic payment orchestration for the Playlist platform: payment sessions, payment links,
allocations/ledger, PSP state machines, webhook/charge-event fulfillment.
- Stack: .NET 10, PostgreSQL, four Lambdas, SNS/SQS with transactional outbox, Stripe/GP/Xendit.
- AutoPay: **not in the AutoPay path** (verified in the HLD sources) — it has its own recurring
  line-item support for sessions it originates.
- Key docs: README.md, AGENTS.md, docs/SETUP.md.
