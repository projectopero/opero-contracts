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
and reference feeds — is maintained separately. This repository is the source
for the `AttestationAnchor` instance listed below. The other graph addresses are
included so a reviewer can follow their on-chain relationships; their presence
does not imply that their source lives in this repository.

## Robinhood Chain Public V2 deployment

The first Public V2 graph is deployed on Robinhood Chain mainnet (chain ID
`4663`). Deployment proves that the contracts and their ownership relationships
exist on chain. It does not by itself mean that an account is funded or that
live trading is available; current availability is published on the
[status page](https://projectopero.com/status).

| Component | Address |
| --- | --- |
| Execution registry | [`0x6DcAE761c0182F441bDDADed27d4b325E176bd23`](https://robinhoodchain.blockscout.com/address/0x6DcAE761c0182F441bDDADed27d4b325E176bd23) |
| Cohort factory | [`0xF03081717aa9e3d0dBe98500a9d14113B0c3B25a`](https://robinhoodchain.blockscout.com/address/0xF03081717aa9e3d0dBe98500a9d14113B0c3B25a) |
| Sequencer feed | [`0x6fa3917Ac833EF34844b8079A6a1E8abFA921E69`](https://robinhoodchain.blockscout.com/address/0x6fa3917Ac833EF34844b8079A6a1E8abFA921E69) |
| Market reference feed | [`0x6B22A786bAa607d76728168703a39Ea9C99f2cD0`](https://robinhoodchain.blockscout.com/address/0x6B22A786bAa607d76728168703a39Ea9C99f2cD0) |
| First vault | [`0x3306559d54c62311789769Ad839A8845DA77FbD0`](https://robinhoodchain.blockscout.com/address/0x3306559d54c62311789769Ad839A8845DA77FbD0) |
| Risk manager | [`0xfa8e535FCc3b641Bf7a68696ADE58e140C3b4A4a`](https://robinhoodchain.blockscout.com/address/0xfa8e535FCc3b641Bf7a68696ADE58e140C3b4A4a) |
| Spot adapter | [`0x66BDE213b830057D4A99d81769f9871D460c6733`](https://robinhoodchain.blockscout.com/address/0x66BDE213b830057D4A99d81769f9871D460c6733) |
| Attestation anchor | [`0xB63C7409F1059d9a5Fe95C4e5b7341C7a5E55cCd`](https://robinhoodchain.blockscout.com/address/0xB63C7409F1059d9a5Fe95C4e5b7341C7a5E55cCd) |

The anchor's immutable publisher is the first vault. The graph was created in
[transaction `0xde3821…6930`](https://robinhoodchain.blockscout.com/tx/0xde3821cfc83e68b78a4597e981cbf1cddd6551e68571dcaddc816d3a04426930),
and its deployment receipt identity is
`sha256:679563d8a79094400d9faf77869e4c81b5302de26a7d5fb7ba955ea6293d706f`.
The complete machine-readable record is in
[`deployments/robinhood-mainnet-public-v2.json`](deployments/robinhood-mainnet-public-v2.json).

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
