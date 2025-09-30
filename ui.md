# Backend & UI

The `ui/` project is both the public website/API and the production
backend that keeps Doot’s state fresh. The backend is implemented as a
set of cron workers living inside the project alongside Next.js pages.

## Responsibilities

- Aggregate prices from many providers every 10 minutes
- Sign results with the oracle key and cache them in Redis
- Pin canonical objects to IPFS and rotate historical CIDs
- Update Zeko L2 (and optionally Mina L1) with new commitments and settle OffchainState
- Serve a fast `/api/get/price` endpoint for SDK consumers

## Cron Workers

- `cron-update-all-prices`
  - Fetches prices from ~20 providers for each token
  - Removes outliers via MAD, computes a mean, signs with the oracle signer (internal secret)
  - Caches per‑token objects (`TOKEN_TO_CACHE`), plus per‑token graph slices
  - Schedule: every 10 minutes (`*/10 * * * *`)

- `cron-update-historical`
  - Reads all cached token objects, merges into prior IPFS state,
    pins a new JSON object, updates `HISTORICAL_CID_CACHE`, and unpins old
  - Schedule: every 10 minutes offset (`5-59/10 * * * *`)

- `cron-zkapp-update-doot-zeko`
  - Reads token caches, constructs canonical IPFS object, pins it,
    derives Merkle root commitment, and updates `Doot` on Zeko L2
  - Generates a settlement proof and calls `settle()`
  - Schedule: every 10 minutes at minutes 2,12,22,32,42,52 (`2-59/10 * * * *`)
  - Confirms transaction with polling; optional 30s delay before settlement

- `cron-zkapp-update-doot-mina`
  - Same as above but against Mina L1; used when you want L1 parity
  - Schedule: every 20 minutes at minutes 2,22,42 (`2-59/20 * * * *`)

## Public API

- `GET /api/get/price?token=<name>`
  - Auth: `Authorization: Bearer <DOOT_API_KEY>`
  - Rate‑limited via Upstash Ratelimit
  - Reads `TOKEN_TO_CACHE[token]` and `TOKEN_TO_AGGREGATION_PROOF_CACHE[token]`
  - Returns `{ price_data, proof_data }` that matches the SDK shape

- `POST /api/verify/verifyAggregated`
  - Verifies a signature over the aggregated price

- `POST /api/verify/verifyIndividual`
  - Verifies a signature from an individual provider sample

## Caches and Keys (Upstash Redis)

- `TOKEN_TO_CACHE[token]` — latest per‑token object
- `TOKEN_TO_GRAPH_DATA[token]` — pre‑computed graph slices and stats
- `TOKEN_TO_AGGREGATION_PROOF_CACHE[token]` — zk aggregation artifact
- `HISTORICAL_CID_CACHE` — CID of rolling historical object
- `ZEKO_CID_CACHE` / `MINA_CID_CACHE` — latest pinned CID + commitment

## IPFS Object

- Built by `pinMinaObject()` with:
  - `assets`: per‑token objects (price, signatures, timestamps, urls)
  - `merkle_map`: `{ commitment, keys, values, witnesses, pinnedAt }`
- Pinned through Pinata with JWT auth and verified via gateway fetch

## On‑Chain Updates

- Contracts compiled once at startup to reduce latency
- Update flow:
  1. Build `TokenInformationArray` in strict token order
  2. Call `update(COMMITMENT, IPFS_HASH, tokenInfo)`
  3. Create `offchainState.createSettlementProof()`
  4. Call `settle(proof)`
  5. Store CID + commitment in Redis

## Environment Variables

Public variables used by the website and SDK:
- `NEXT_PUBLIC_PINATA_GATEWAY` — gateway host used to fetch pinned IPFS JSON
- `NEXT_PUBLIC_ZEKO_DOOT_PUBLIC_KEY` — Zeko L2 contract public key
- `NEXT_PUBLIC_MINA_DOOT_PUBLIC_KEY` — Mina L1 contract public key (if used)

Internal credentials (not documented publicly) back the cron workers and API
and include Redis credentials, an oracle signer, IPFS pinning token, and optional
provider API keys. Operators should manage these via a secret manager.

See Configuration for the minimal public list.

## Operational Notes

- Cron order matters: update‑all‑prices → update‑historical → zeko/mina
- If OffchainState reads fail with `Field.assertEquals`, wait and retry – the
  state is still settling
- Large proof generation or network hiccups are handled with polling and
  time‑bounded waits to avoid stuck jobs
