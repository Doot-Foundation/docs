# Runbook

Operational playbooks for running Doot in production.

## Health and Status

- API health: `GET /api/health` on the UI service
- Redis connectivity: verify your Redis service credentials are valid and reachable
- Latest IPFS CIDs: `HISTORICAL_CID_CACHE`, `ZEKO_CID_CACHE`, `MINA_CID_CACHE`
- On‑chain status: query contract accounts on Zeko/Mina explorers

## Common Tasks

- Force a fresh cache for a token
  - Call `GET /api/reset/resetEveryCache` (admin‑only in production)
- Roll new historical CID immediately
  - Trigger `cron-update-historical` job
- Re‑update chain after transient error
  - Trigger `cron-zkapp-update-doot-zeko` (and/or mina)

## Incident Response

- API failures
  - SDK automatically falls back to L2/L1; raise priority if both chain reads also fail
  - Verify Redis availability, rate limit status, and that provider keys are valid

- Chain update stuck with Field.assertEquals
  - OffchainState still settling; wait 20–60 seconds and retry

- IPFS pinning fails
  - Verify the IPFS pinning credentials in your deployment environment, gateway reachability, and that the JSON body is under size limits

## Keys and Rotation

- The oracle signer key is used to sign aggregated prices and send transactions
- Rotate by updating secrets in your environment; update Registry if contract ownership changes

## Observability Suggestions

- Emit structured logs from cron jobs with duration, token counts, and CID values
- Monitor: job duration, failure rate, Redis error rate, API latency, 429 rate
