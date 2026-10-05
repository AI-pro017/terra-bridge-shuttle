# Terra Bridge Shuttle

![Shuttle Banner](/resources/banner.png)

A token bridge between Terra and EVM chains, based on Terraform Labs' Shuttle and extended with a Fantom testnet route for FantOHM's FHM token.

When you send a supported token to the shuttle address on Terra, a wrapped version is minted to your address on the other chain. When you burn the wrapped token on the EVM side, the original is released back to you on Terra. A small relaying fee is taken on each transfer.

## Supported networks

The relayers are configured for Ethereum (mainnet and Ropsten), BNB Chain, Harmony (each with a testnet) and the Fantom testnet. Assets include LUNA, UST, KRT, SDT, MNT, MIR, ANC, aUST and Mirror's mAssets, plus FHM on Fantom. FHM uses 9 decimals instead of 18, which both relayers handle as a special case.

## How it's put together

| Folder | What it does |
| --- | --- |
| `contracts/` | Solidity contracts for the EVM side: an ERC20 `WrappedToken` for each asset, a `Minter` that mints and burns them, and a `ShuttleVault`. Deployed with Truffle. |
| `terra/` | Relayer that watches Terra for deposits to the shuttle address and mints the wrapped token on the EVM chain. It also checks prices through an oracle to work out fees. |
| `eth/` | Relayer that watches the EVM chain for burns of wrapped tokens and sends the original asset back on Terra. |
| `fee/` | A job that compares wrapped supply with the shuttle's balance and moves collected fees to the fee collector address. |
| `form-collector/` | A small HTTP service with a `/recover` endpoint for queueing a transaction hash to be relayed again. |

The relayers keep their progress in Redis and DynamoDB (or MongoDB), and can post alerts to Slack.

## Running the relayers

Each service is a separate Node.js app with its own config:

```bash
git clone https://github.com/AI-pro017/terra-bridge-shuttle.git
cd terra-bridge-shuttle/terra
cp .env_example .env
npm install
npm start
```

Do the same in `eth/`, `fee/` and `form-collector/`. The `.env_example` in each folder lists its settings, mainly the Terra and EVM RPC URLs and chain IDs, the mnemonics of the relayer wallets, fee settings, and the Redis, DynamoDB and Slack connections. Token addresses for each network are in each service's `src/config/` folder.

## Deploying the contracts

```bash
cd contracts
cp .env_example .env
npm install
npx truffle migrate --network <network>
```

## Status

This was built for the original Terra chain (`columbus-5`, now Terra Classic). Most of the networks above, including Ropsten and the old Terra endpoints, have since been shut down, so running it today would need new RPCs and redeployed contracts.

## License

MIT, from the original Shuttle code by Terraform Labs. See `eth/LICENSE` and `fee/LICENSE`.
