# Chio Showcase: Internet of Agents Web3 Network

Flagship standalone demo for Chio-mediated agent commerce over the Chio web3
evidence stack. This repository is self-contained: it depends only on the
published `chio` binary and the vendored helpers under `scripts/`. No sibling
checkout of the Chio monorepo is required.

This example keeps the service-order story from the original web3 bundle, but
promotes it into a four-organization topology where every default cross-org
API or tool boundary is mediated by Chio:

- Atlas Operator runs the treasury, procurement, settlement, and auditor agents.
- ProofWorks Provider runs the provider agent and provider review tool.
- CipherWorks Review Lab runs a specialist subcontractor agent behind its own
  trust-control authority and MCP edge.
- Meridian Federation Verifier evaluates passport, reputation, and federation
  admission evidence.

The scenario uses local services and local receipts by default. Base Sepolia
evidence is attached and verified when the live smoke report already exists, but
the example does not send chain transactions unless an operator separately runs
the deployment or smoke scripts. Mainnet remains blocked.

## Scenario

A treasury agent delegates a bounded web3 service budget. A procurement agent
runs a three-provider RFQ through a market broker protected by
`chio api protect`, rejects a cheap low-reputation provider and a malicious
over-budget provider, then selects ProofWorks through Chio policy over passport,
reputation, budget, runtime tier, and federation admission. ProofWorks
subcontracts a specialist proof-leaf review to CipherWorks through a narrowed
two-hop capability. A settlement agent routes the payment rail, assembles the
packet, maps it into a Chio web3 settlement dispatch, and an auditor verifies the
bundle through a read-only web3 evidence MCP edge.

The same run also exercises the control-plane surfaces that make this a Chio
example instead of a direct HTTP demo:

- `chio trust serve` issues root, delegated, provider, subcontractor,
  federation, and sidecar capabilities with runtime-attestation and SPIFFE
  workload identity evidence.
- `chio api protect` mediates the market broker and settlement desk OpenAPI
  services and emits sidecar receipts.
- `chio mcp serve-http` mediates the web3 evidence, provider review, and
  subcontractor review tools.
- Passport, challenge, presentation, reputation, federation policy, evidence
  export/import, and federated issuance artifacts are produced by `chio`
  CLI/trust-control calls and carry explicit source provenance in the bundle.
- Negative controls prove invalid SPIFFE identity, overspend, and velocity burst
  denials, each with a denial receipt.
- The default smoke also proves signed human approval, x402-style payment proof,
  cross-rail settlement routing, runtime trust degradation and re-attestation,
  receipt-correlated operations telemetry, historical reputation drift, and six
  adversarial denials.

## Prerequisites

- The `chio` binary on `PATH` (or `CHIO_BIN` pointing at a local build).
  Install with:
  ```bash
  curl -fsSL https://www.chio.world/install.sh | sh
  ```
- Python 3.11+ with [uv](https://docs.astral.sh/uv/).
- Bun 1.3+ (only required for the Evidence Console UI and its Playwright
  e2e suite).

## Running

Sync Python dependencies once:

```bash
uv sync
```

Run the mediated scenario end-to-end:

```bash
./smoke.sh
```

The smoke writes a reviewable artifact bundle under `artifacts/web3-service-order/<timestamp>/`
and asserts the verifier returns `ok: true`.

To pin the output directory:

```bash
./smoke.sh --artifact-dir "$(pwd)/artifacts/demo"
```

Base Sepolia evidence is attached when `target/web3-live-rollout/base-sepolia/base-sepolia-smoke.json`
exists inside the example root (produced by a separate live rollout run).
To require that evidence and fail if missing:

```bash
./smoke.sh --require-base-sepolia-smoke
```

Mainnet is blocked by policy.

### Evidence Console (Next.js UI)

The `app/` directory is a Next.js 15 reviewer UI that renders an artifact
bundle. It performs in-browser SHA-256 verification on every fetched file and
fails closed when the bundle is missing or tampered.

Run the full scenario plus the Playwright e2e suite:

```bash
CHIO_RUN_E2E=1 ./smoke.sh
```

Or start the app standalone against any bundle directory:

```bash
cd app
bun install
bun run build
CHIO_BUNDLE_DIR="$(pwd)/tests/fixtures/good-bundle" bun run start
```

See `app/README.md` for the full bundle-schema contract and env vars.

## Structure

```text
internet_web3/artifacts.py   evidence-bundle store and manifest writer
internet_web3/adversarial.py adversarial denial controls
internet_web3/approval.py    deterministic signed human approval fixture
internet_web3/budgeting.py   budget exposure, reconciliation, and overspend denial
internet_web3/capabilities.py deterministic Ed25519 identities and local delegation links
internet_web3/chio.py        trust-control and Chio MCP HTTP clients
internet_web3/chio_cli.py    Chio CLI backed passport, reputation, federation workflows
internet_web3/clients.py     API sidecar and MCP edge adapters
internet_web3/disputes.py    partial payment, refund, remediation, and dispute audit
internet_web3/federation.py  legacy local federation dataclasses and helpers
internet_web3/identity.py    SPIFFE, runtime appraisal, and degradation workflows
internet_web3/marketplace.py RFQ selection, history, and x402 payment handshake
internet_web3/observability.py trace, SIEM, and operations timeline artifacts
internet_web3/rails.py       Base Sepolia, local devnet, and Solana proof rail routing
internet_web3/reports.py     topology, receipt, behavior, and guardrail reports
internet_web3/scenario.py    service-order application coordinator
internet_web3/subcontracting.py two-hop subcontractor delegation and review
internet_web3/verify.py      offline bundle verifier
orchestrate.py               CLI entrypoint
policies/                    MCP edge policies and API-sidecar policy notes
services/                    raw FastAPI services plus OpenAPI sidecar specs
tools/                       web3 evidence, provider review, and subcontractor MCP servers
scenario/                    topology bootstrap scripts
scripts/common.sh            vendored shell helpers (chio-bin resolution, port picking)
workspaces/                  operator and provider fixture state
app/                         Next.js Evidence Console + Playwright e2e
smoke.sh                     smoke runner (pass CHIO_RUN_E2E=1 for UI e2e)
```

The raw FastAPI services are implementation details. The scenario receives only
the Chio sidecar URLs for default execution, and the verifier fails if the
topology reports a direct unmediated default path.

## Artifact Contract

The smoke writes a reviewable bundle under the selected artifact directory:

```text
agents/                      deterministic agent decisions
behavior/                    behavioral feed, baseline, and pass status
adversarial/                 prompt, invoice, replay, expiry, rail, and passport denials
approvals/                   signed high-risk release approval challenge and receipt
capabilities/                local delegated capability projections
chio/topology.json           four-org mediated runtime topology
chio/capabilities/           trust-control-issued capabilities
chio/receipts/               trust, API sidecar, MCP, and lineage receipt summaries
chio/budgets/                exposure authorization and spend reconciliation
contracts/                   service order, settlement packet, dispatch, receipt
disputes/                    weak deliverable, partial payment, refund, remediation
evidence/                    read-only web3 evidence MCP output
federation/                  Chio policy, export/import, admission, federated cap
financial/                   settlement reconciliation
guardrails/                  invalid SPIFFE, overspend, and velocity denial receipts
identity/passports/          Chio provider passport, provenance, and verdicts
identity/presentations/      Chio verifier challenge, holder presentation, verdict
identity/runtime-appraisals/ workload runtime assurance fixtures
identity/runtime-degradation/ quarantine and re-attestation flow
lineage/                     delegated capability chain projections
market/                      RFQ, bids, selection, quote, and fulfillment package
operations/                  trace map, SIEM events, operations timeline
payments/                    x402 payment-required and Chio payment proof artifacts
provider/                    provider review and reputation evaluation artifacts
reputation/                  Chio local report, passport comparison, admission verdict
scenario/                    copied order, policy, catalog, and timeline
settlement/                  cross-rail settlement selection rationale
subcontracting/              specialist review capability, obligations, attestation
web3/                        copied validation ladder and optional Base Sepolia evidence
bundle-manifest.json         SHA-256 manifest for offline review
review-result.json           verifier verdict
summary.json                 operator-facing pass/fail summary
```

`review-result.json` fails closed when required Chio artifacts are missing, any
default path is unmediated, a denial control does not deny, budget spend is not
reconciled, RFQ routing admits the wrong provider, two-hop subcontractor lineage
is missing, approval/payment/rail/dispute/runtime/observability checks fail, any
adversarial control allows, or a required Base Sepolia attachment is incomplete.

## What It Proves

- Recursive agent commerce can use x402-style payment requirements without
  treating x402 as the settlement source of truth.
- Procurement, provider execution, settlement, audit, and federation authority
  are separated and mediated by Chio capability tokens.
- API and MCP boundaries run through Chio sidecars or Chio MCP HTTP edges, not
  direct HTTP or stdin in the default path.
- Chio can route a provider market using trust, budget, runtime, federation, and
  reputation instead of acting only as an allow/deny wrapper.
- Two-hop subcontracting can inherit obligations and preserve receipt lineage.
- High-risk release requires a signed approval artifact before budget exposure
  and payment proof.
- Cross-rail settlement can choose Base Sepolia when evidence exists, fall back
  to local devnet otherwise, and deny unsupported Solana memo settlement in the
  same review bundle.
- Runtime degradation, quarantine, and re-attestation are reviewable artifacts.
- Operations telemetry correlates business events, Chio boundaries, and receipt
  identifiers.
- Prompt injection, invoice tampering, quote replay, expired capability reuse,
  unauthorized rail routing, and forged passport attempts all deny.
- Provider admission is tied to passport presentation, local reputation,
  federation policy, runtime appraisal, and SPIFFE workload identity artifacts.
- Budget exposure is authorized before quote acceptance and reconciled after
  settlement packet assembly.
- Behavioral baseline artifacts are produced from the Chio behavioral-feed
  model without overclaiming HushSpec runtime deny wiring.
- Optional Base Sepolia smoke evidence includes real tx hashes for operator
  setup, USDC approval, escrow create, root publication, release/refund paths,
  and price readback when the public testnet smoke report exists.
