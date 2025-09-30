# L1/L2 Contracts

How the on‑chain layer stores price data and why OffchainState is used.

## Doot zkApp
- Storage:
  - `commitment: Field` — Merkle root of the canonical IPFS object
  - `ipfsCID: IpfsCID` — packed CID string (via `o1js-pack`)
  - `owner: PublicKey` — admin for upgrades
  - `offchainStateCommitments` — commitments for the OffchainState
- OffchainState:
  - `tokenInformation: Map<Field, TokenInformationArray>`
  - `TokenInformationArray = { prices: Field[10] }`
  - We always read/write index `Field(0)` with the 10‑token vector

### Methods

- `initBase(updatedCommitment, updatedIpfsCID, informationArray)`
  - Bootstraps the contract once, sets owner, commitment, CID, and off‑chain vector
- `update(updatedCommitment, updatedIpfsCID, informationArray)`
  - Owner‑gated update of commitment + CID and off‑chain vector
- `settle(proof)`
  - Applies an OffchainState settlement proof after `update()` to bind the vector
- `getPrices(): TokenInformationArray`
  - Returns the 10‑token vector (order defined in Architecture → Token Ordering)
- `verify(signature, deployer, price)`
  - Stand‑alone signature check utility

### Why OffchainState?

- The full set of prices and proofs would be expensive to store directly on‑chain
- OffchainState provides canonical commitments + merkleized state with on‑chain verification via settlement proofs
- The IPFS object (CID) holds the full material: witness paths, values, and metadata

## Networks

- Zeko L2 (devnet): fast finality, same API for `o1js`
- Mina L1 (devnet or mainnet): maximum security and decentralization

Operational updaters submit transactions to these networks on a schedule; see Backend & UI for timing.

## Aggregation Programs
- ZkPrograms that prove aggregate summaries for 20‑ or 100‑length price vectors
- Used by backend jobs to create verifiable aggregation artifacts (cached separately)

## Registry Contract
- Purpose: record latest implementation address and code references
  - GitHub source link (packed string)
  - IPFS source link (packed string)
  - Latest implementation `PublicKey`
- Methods: `initBase()`, `upgrade(updatedGithubLink, updatedIPFSLink, updatedImplementation)`

## Deployment Notes

- Always compile OffchainState and contract before first use
- Pass prices in the correct fixed order
- After `update()`, create and submit `settle()` proof to commit the OffchainState
- Store the produced CID in Redis to enable UI and SDK linking
