# Chain Sandbox

Deterministic local blockchain nodes for development and CI.

`chain-sandbox` starts disposable development chains behind one tiny interface. Required tests never depend on public RPCs, faucets, shared devnets, or third-party credentials.

## Supported chains

| Chain | Backend | RPC |
| --- | --- | --- |
| `bitcoin` | Bitcoin Core 31.1, regtest | `BITCOIN_RPC_URL` |
| `ethereum` | Anvil / Foundry 1.8.0 | `ETHEREUM_RPC_URL` |
| `solana` | Agave 4.2.1 local validator | `SOLANA_RPC_URL` |

Versions are pinned in [`versions.env`](versions.env). The Agave release archive is checksum-verified before execution.

> The `bitcoin/bitcoin` container is explicitly intended for testing environments and is not an official Bitcoin Core distribution. Chain Sandbox never uses it for production custody.

## GitHub Actions

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: Pom4H/chain-sandbox@main
    with:
      chain: bitcoin

  - run: cargo test -p my-bitcoin-integration-tests
    env:
      BITCOIN_RPC_URL: ${{ env.BITCOIN_RPC_URL }}
```

Use a commit SHA or release tag instead of `main` in production CI.

The action starts the node in the **same job** as the consumer tests. This is intentional: a reusable workflow would run in another job and could not expose its local node process to the caller.

## CLI

```bash
./bin/chain-sandbox start bitcoin
./bin/chain-sandbox health bitcoin
./bin/chain-sandbox stop bitcoin

./bin/chain-sandbox start ethereum
./bin/chain-sandbox start solana
```

Environment variables let callers override default ports:

```text
CHAIN_SANDBOX_BITCOIN_PORT=18443
CHAIN_SANDBOX_ETHEREUM_PORT=8545
CHAIN_SANDBOX_SOLANA_PORT=8899
```

Runtime data lives under `${CHAIN_SANDBOX_HOME:-$HOME/.cache/chain-sandbox}`. Node state is reset on every `start`; downloaded tool archives are cached.

## CI model

Run each chain in a separate job. Do not start all chains for core-only changes. Public testnet/devnet checks should be scheduled compatibility smoke tests and should not gate pull requests.

The repository self-tests the actual RPC boundary for every backend:

- Bitcoin: `getblockchaininfo`, wallet creation, address generation and 101 regtest blocks;
- Ethereum: `eth_chainId`, `eth_blockNumber`, deterministic funded accounts;
- Solana: `getHealth` and `getLatestBlockhash`.

## Contract

Chain Sandbox owns installation, process/container lifecycle, readiness and version pinning. Consumers own chain-specific business assertions.

```text
consumer test
    │
    ▼
*_RPC_URL
    │
    ▼
Chain Sandbox
    │
    ├── Bitcoin Core regtest
    ├── Anvil
    └── Agave local validator
```

MIT.
