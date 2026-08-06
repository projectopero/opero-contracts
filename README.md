# opero-contracts

On-chain receipt anchoring and execution-safety contracts for
[Opero AI](https://github.com/projectopero).

An agent that reports its own results is asking to be trusted. These contracts
remove that ask for the part that matters: what the agent committed to, and when.
The commitment is a Merkle root over a batch of trade records, appended in strict
sequence and never rewritable. Anyone can take a published batch, recompute the
root with [opero-protocol](https://github.com/projectopero/opero-protocol), and
compare it to what is on chain.

The contract stores only the root, so publishing the underlying records stays the
agent's choice. What it cannot do is change what it already committed, and
withholding is visible — a root with no published batch is a gap anyone can see.

## Contracts

| Contract | Responsibility |
| --- | --- |
| `AttestationAnchor` | Append-only registry of Merkle roots over trade-log batches. One publisher, strictly increasing sequence, no rewrites. |
| `SequencerGate` | Pins a Chainlink-shaped feed to the exact code hash of its source, so the feed behind a consumer cannot be swapped for a different implementation after the fact. |

`AttestationAnchor` is deliberately small. It has one writer, one operation, no
upgrade path and no owner — the properties it needs are the ones you can read off
it in a minute.

## What is not here

The strategy execution graph — vaults, factories, risk manager, venue adapters
and reference feeds — is not published. It encodes the position logic, and its
parameters are enforced and readable on chain regardless, so publishing the
source would give away the approach without adding anything a reviewer cannot
already verify against the deployed bytecode.

Deployment addresses are published with each finalized release rather than in
advance. Nothing here is deployed to a production address yet.

## Build and test

```sh
forge build
forge test
```

`forge-std` is a pinned submodule; clone with `--recurse-submodules` or run
`git submodule update --init --depth 1`.

## Verifying an anchored batch

```sh
git clone https://github.com/projectopero/opero-protocol
cd opero-protocol && npm install
```

```js
import { verifyAgainstChain } from "./src/onchain.mjs";

const result = await verifyAgainstChain({
  rpc: "https://your-rpc",
  anchor: "0x…",          // deployed AttestationAnchor
  sequence: 1,
  records: publishedRecords,
});
// result.ok is true only if the published records reproduce the on-chain root
```

## Security

Report vulnerabilities per the organization
[security policy](https://github.com/projectopero/.github/blob/main/SECURITY.md).
Please do not open a public issue for a suspected vulnerability.

## License

Apache-2.0. See [LICENSE](LICENSE).
