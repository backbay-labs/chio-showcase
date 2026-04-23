# Internet of Agents Incident Network

Multi-org incident response using Chio for capability-governed tool access.
Every tool call goes through the Chio kernel (`chio mcp serve-http`), which
evaluates guard policies and signs receipts.

## Scenario

Meridian Labs has a sev-1 outage on their inference gateway. A bad edge rule
(`geo-restrict-v42`) from their CDN provider Stratos Networks is blocking
legitimate traffic. A commander agent orchestrates investigation, delegates
a bounded fix to the provider, all under Chio capability governance.

## Architecture

```
chio trust serve                     capability authority
chio mcp serve-http  x4              kernel-mediated MCP tool access
services/acp_broker.py              cross-org task broker
services/coordinator.py             provider entry point
services/executor.py                bounded operation runner
services/approval.py                approval token issuance
orchestrate.py                      agent orchestrator (entry point)
```

Agents use the OpenAI Agents SDK (`Agent`, `Runner`, `FunctionTool`) or
Anthropic SDK for tool-use loops. Each MCP tool call is wrapped as a
`FunctionTool` backed by a Chio MCP endpoint.

## Running

```bash
# Install the chio CLI (once, if not already on PATH)
curl -fsSL https://www.chio.world/install.sh | sh

# Install Python deps from PyPI into a uv-managed venv
uv sync

# Run the smoke test
./smoke.sh
```

Set `OPENAI_API_KEY` for live agent reasoning, or run without for
deterministic fallback (CI mode). If `chio` is already installed to a
non-standard location, export `CHIO_BIN=/path/to/chio` before running.

## Scenarios

| Script | Tests |
|--------|-------|
| `scenario/01-happy-path.sh` | Full 6-hop delegation, evidence bundle |
| `scenario/02-attenuation-deny.sh` | Executor exceeds scope, denied |
| `scenario/03-revoke-midchain.sh` | Upstream revocation propagates |
| `scenario/04-approval-required.sh` | Broader rollback needs approval |
| `scenario/05-expiry-async-failure.sh` | Capability TTL expires |

## Structure

```
incident_network/           Python package
  chio.py                   Chio MCP client, trust-control client
  capabilities.py           Identity, delegation, signing
  agents.py                 Agent definitions, prompts, runner
  verify.py                 Bundle verification
orchestrate.py              Entry point
services/                   FastAPI services (separate processes)
tools/                      MCP servers (stdio, wrapped by Chio)
policies/                   HushSpec guard policies
workspaces/                 Fixture data
scenario/                   Scenario runners
```
