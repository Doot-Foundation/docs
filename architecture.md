# Architecture

This section explains how data moves through Doot and how the
components fit together. It is the best place to get a mental model
before touching code or infra.

## High‑Level Components

- SDK (`npm/main`)
  - Provides a simple `Client` with a smart fallback: API → L2 → L1
  - Normalizes the response shape across all sources
  - Caches compilation and recent on‑chain reads to reduce latency

- Backend (“UI” project) (`ui/`)
  - Next.js service that also hosts the production cron workers
  - Cron workers fetch prices from many providers, aggregate, sign,
    cache in Upstash Redis, pin objects to IPFS (Pinata), and update
    the on‑chain OffchainState on Zeko L2 and Mina L1
  - Public API (`/api/get/price`) for fast reads with API keys

- Smart Contracts (`contracts/` and SDK `src/constants/Doot.ts`)
  - `Doot` zkApp stores an OffchainState commitment and an IPFS CID
  - `getPrices()` returns the full vector of 10 token prices
  - `update()` moves the commitment + CID forward; `settle()` commits
    the OffchainState proof; `initBase()` bootstraps the contract

- Storage and Infra
  - Upstash Redis: price cache, graph cache, proof cache, current IPFS CIDs
  - IPFS (Pinata): canonical object for the latest aggregation state,
    including Merkle root, witnesses, and per‑token data
  - Zeko L2 + Mina L1: on‑chain anchoring of OffchainState commitment

## Data Flow (10‑Minute Cycle)

1) Fetch + Aggregate
- Cron `update-all-prices` hits ~20 providers per asset
- Cleans data using Median Absolute Deviation (MAD) to remove outliers
- Computes mean price, signs it with the oracle signer (internal secret), caches
  per‑token objects to Upstash, and pre‑builds graph slices

2) Persist History
- Cron `update-historical` reads all token caches, merges into the
  previously pinned historical object, pins the new version to IPFS,
  and rotates the historical CID stored in Redis

3) On‑Chain Update (Zeko L2, then Mina L1 as needed)
- Cron `cron-zkapp-update-doot-zeko` (and the Mina counterpart) reads
  cached token objects, constructs a canonical IPFS object, pins it,
  derives its Merkle root commitment, and calls `update()` on the
  `Doot` contract; then generates and submits a `settle()` proof

4) SDK Consumption
- The public API serves the freshest cache with millisecond latency
- If API is down, the SDK reads from Zeko L2 (tens of seconds)
- If L2 is down, the SDK reads from Mina L1 (tens of seconds to a minute)

## Token Ordering and Indexing

On‑chain `getPrices()` returns a struct of 10 `Field` values in a fixed order.
The SDK maps friendly token names to indexes as follows:

- 0: mina
- 1: bitcoin
- 2: ethereum
- 3: solana
- 4: ripple
- 5: cardano
- 6: avalanche
- 7: polygon
- 8: chainlink
- 9: dogecoin

Keep this ordering consistent across aggregation, IPFS object building,
and contract calls.

## Trust Model

- API: fast and signed data from Doot’s infra + cache
- L2: fast on‑chain source anchored on Zeko with quick finality
- L1: slowest but most secure (Mina mainnet or devnet)

All sources return a normalized shape so that consuming applications can
switch without changes. Proof material and signatures are included or
linkable (via CID) for independent verification.
