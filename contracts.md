# Doot L1 Mina Smart Contracts

Adapted from `contracts/README.md`.

This repository tracks the Mina smart contracts under Doot – a data feeds oracle for Mina Protocol.

## Registry.ts

Indexes the last updated implementation details:

1. Source code at GitHub
2. Source code pinned at IPFS
3. Address of the latest implementation

Developers can refer to this contract to check for changes to Doot smart contracts.

## Doot.ts

Core contract responsible for bringing the current exchange rate on-chain (currently updated every 2 hours).

Developers can call `getPrice(token: CircuitString)` to fetch the exchange rate of a tracked cryptocurrency and use it in their smart contracts.

## AggregationProgram.ts

Creates aggregation proofs for each asset update, enabling a verifiable aggregation process. This general `ZkProgram` is used directly in price generation cron jobs.

## Barter

Two parts:

- Mina: everything Mina Protocol
- Ethereum: everything EVM

