# Doot Protocol Documentation

This is the canonical, production‑grade documentation for the Doot Oracle.
It explains how the system works end‑to‑end, how to run it, and how to
integrate it safely in production.

Doot provides signed, verified price feeds for 10 core crypto assets with
multiple levels of trust and availability:

- API (fastest): served from the Doot backend cache
- L2 (Zeko): on‑chain reads with fast finality
- L1 (Mina): on‑chain reads with maximum security

The SDK automatically falls back API → L2 → L1, returning a consistent
response shape regardless of source.

What’s inside these docs:

- Architecture: Components and data flow
- SDK: Usage, methods, response formats, fallbacks, and errors
- Backend: Cron jobs, caching, aggregation, IPFS pinning, and on‑chain updates
- Contracts: L1/L2 design, off‑chain state, methods, and deployment notes
- Operations: Config, runbooks, monitoring, and troubleshooting
- Examples: Practical usage and reference snippets

New to Doot? Start with Architecture, then pick the section for your role
(SDK consumer vs. protocol operator).
