# NexLend
**Fractionally Collateralized Debt Engine on Stellar**

NexLend is a decentralized lending protocol built on **Stellar using Soroban smart contracts**. It enables users to lock unique on-chain assets — such as tokenized real estate or invoice tokens — as collateral to mint **xUSD**, a protocol-native synthetic stablecoin. All collateralization ratios, liquidation thresholds, and auction mechanics are enforced fully on-chain with no custodial intermediaries.

The project solves the problem of illiquid real-world assets sitting idle on-chain by transforming them into productive collateral for synthetic liquidity. NexLend is designed for developers, DeFi builders, and financial communities interested in composable lending infrastructure backed by real-world asset primitives on a low-fee, fast-finality blockchain.

---

## 🚀 Core Features

- Non-custodial vault system via Soroban smart contracts
- Accepts unique on-chain assets as collateral (tokenized real estate, invoice tokens, RWA NFTs)
- Mints **xUSD** — a protocol-native synthetic stablecoin — against locked collateral
- Dynamic collateralization ratio tracking powered by on-chain price feeds
- Automatic liquidation threshold monitoring
- Public Dutch auction system for undercollateralized vaults
- Front-run protection and strict security checks for liquidators
- Complex on-chain math for real-time collateral valuation
- Web interface for vault management and auction participation

---

## 🏗 Architecture Overview

- **Frontend (`apps/web`)**
  Next.js application for interacting with NexLend smart contracts. Provides user interface for opening vaults, depositing collateral, minting xUSD, monitoring collateral health, and participating in Dutch auctions.

- **Backend (`apps/api`)**
  Node.js API for off-chain services such as indexing vault events, tracking collateralization ratios, sending liquidation alerts, managing user metadata, and aggregating protocol analytics.

- **Smart Contracts (`contracts/`)**
  Soroban smart contracts written in Rust that handle all vault state management, xUSD minting and burning, collateral ratio calculations, liquidation logic, Dutch auction execution, and security enforcement against front-running.

- **Price Oracle (`oracle/`)**
  Off-chain oracle services that feed verified real-world asset price data on-chain to enable accurate collateral valuation and trigger liquidation thresholds when asset values decline.

---

## 📁 Repository Structure

```text
/
├── apps/
│   ├── web/              # Next.js frontend
│   └── api/              # Node.js backend API
├── contracts/            # Soroban smart contracts (Rust)
│   ├── vault/            # Vault and collateral management
│   ├── xusd/             # xUSD synthetic stablecoin contract
│   ├── auction/          # Dutch auction engine
│   └── oracle/           # On-chain price feed consumer
├── oracle/               # Off-chain oracle price feed service
├── packages/             # Shared utilities and types
├── scripts/              # Deployment and automation scripts
├── tests/                # Integration and E2E tests
└── README.md
```

---

## 🛠 Setup Instructions

### Prerequisites

Before you begin, ensure you have the following installed:

- **Node.js** (v18 or higher) - [Download](https://nodejs.org/)
- **npm** or **yarn** - Comes with Node.js
- **Rust** (stable toolchain) - [Install](https://rustup.rs/)
- **Soroban CLI** - Instructions below
- **Stellar testnet account** - We'll create this in setup

### Installation Overview

1. Clone the repository
2. Set up smart contracts
3. Set up backend API
4. Set up price oracle
5. Set up frontend
6. Run tests

---

## 📦 1. Clone the Repository

```bash
git clone https://github.com/your-org/nexlend.git
cd nexlend
```

---

## 🔗 2. Smart Contracts Setup (Soroban)

### Install Soroban CLI

```bash
cargo install --locked stellar-cli --features opt
```

Or use the install script:

```bash
curl -fsSL https://github.com/stellar/stellar-cli/raw/main/install.sh | sh
```

Verify installation:

```bash
stellar --version
```

### Configure Stellar Testnet

```bash
stellar network add --global testnet \
  --rpc-url https://soroban-testnet.stellar.org:443 \
  --network-passphrase "Test SDF Network ; September 2015"
```

### Generate Identity & Fund Account

```bash
stellar keys generate --global deployer --network testnet
```

Get your address:

```bash
stellar keys address deployer
```

Fund your account using Friendbot:

```bash
curl "https://friendbot.stellar.org?addr=$(stellar keys address deployer)"
```

Verify balance:

```bash
stellar account balance --id deployer --network testnet
```

### Build Contracts

```bash
cd contracts
cargo build --target wasm32-unknown-unknown --release
```

### Deploy Contracts

Deploy the xUSD stablecoin contract:

```bash
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/nexlend_xusd.wasm \
  --source deployer \
  --network testnet
```

Deploy the vault contract:

```bash
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/nexlend_vault.wasm \
  --source deployer \
  --network testnet
```

Deploy the Dutch auction contract:

```bash
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/nexlend_auction.wasm \
  --source deployer \
  --network testnet
```

Save all three contract IDs — you'll need them for the frontend, backend, and oracle configuration.

### Initialize Contracts

Initialize the xUSD contract:

```bash
stellar contract invoke \
  --id YOUR_XUSD_CONTRACT_ID \
  --source deployer \
  --network testnet \
  -- initialize \
  --admin $(stellar keys address deployer) \
  --vault YOUR_VAULT_CONTRACT_ID
```

Initialize the vault contract:

```bash
stellar contract invoke \
  --id YOUR_VAULT_CONTRACT_ID \
  --source deployer \
  --network testnet \
  -- initialize \
  --admin $(stellar keys address deployer) \
  --xusd YOUR_XUSD_CONTRACT_ID \
  --auction YOUR_AUCTION_CONTRACT_ID \
  --min_collateral_ratio 150 \
  --liquidation_threshold 120
```

Initialize the auction contract:

```bash
stellar contract invoke \
  --id YOUR_AUCTION_CONTRACT_ID \
  --source deployer \
  --network testnet \
  -- initialize \
  --admin $(stellar keys address deployer) \
  --vault YOUR_VAULT_CONTRACT_ID \
  --auction_duration 3600
```

---

## 🖥 3. Backend Setup (Node.js API)

```bash
cd apps/api
npm install
```

### Create Environment File

Create `.env` in `apps/api/`:

```env
PORT=3001
NODE_ENV=development

# Stellar Network
STELLAR_NETWORK=testnet
SOROBAN_RPC_URL=https://soroban-testnet.stellar.org
HORIZON_URL=https://horizon-testnet.stellar.org

# Contracts
VAULT_CONTRACT_ID=YOUR_DEPLOYED_VAULT_CONTRACT_ID
XUSD_CONTRACT_ID=YOUR_DEPLOYED_XUSD_CONTRACT_ID
AUCTION_CONTRACT_ID=YOUR_DEPLOYED_AUCTION_CONTRACT_ID

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/nexlend

# Optional
REDIS_URL=redis://localhost:6379

# Alerts
LIQUIDATION_ALERT_THRESHOLD=130
```

### Run Database Migrations

```bash
npm run migrate
```

### Start Backend Server

```bash
npm run dev
```

Backend should now be running at `http://localhost:3001`

### Verify Backend

```bash
curl http://localhost:3001/health
```

---

## 🔮 4. Price Oracle Setup

```bash
cd oracle
npm install
```

### Create Environment File

Create `.env` in `oracle/`:

```env
PORT=3002
NODE_ENV=development

# Stellar
SOROBAN_RPC_URL=https://soroban-testnet.stellar.org
ORACLE_SIGNER_SECRET=YOUR_ORACLE_ACCOUNT_SECRET_KEY

# Contracts
VAULT_CONTRACT_ID=YOUR_DEPLOYED_VAULT_CONTRACT_ID

# Price Feed Sources
COINGECKO_API_URL=https://api.coingecko.com/api/v3
RWA_PRICE_FEED_URL=YOUR_RWA_PRICE_PROVIDER_URL
FEED_INTERVAL_MS=30000

# Collateral Config
SUPPORTED_ASSET_TYPES=real_estate,invoice_token
```

### Start Oracle Service

```bash
npm run dev
```

Oracle service should now be running at `http://localhost:3002`

---

## 🌐 5. Frontend Setup (Next.js)

```bash
cd apps/web
pnpm install
```

### Create Environment File

Create `.env.local` in `apps/web/`:

```env
NEXT_PUBLIC_BASE_URL=https://nexlend.app
NEXT_PUBLIC_HORIZON_PUBLIC_URL=https://horizon.stellar.org
NEXT_PUBLIC_HORIZON_TESTNET_URL=https://horizon-testnet.stellar.org
NEXT_PUBLIC_STELLAR_NETWORK=testnet
NEXT_PUBLIC_VAULT_CONTRACT_ID=YOUR_DEPLOYED_VAULT_CONTRACT_ID
NEXT_PUBLIC_XUSD_CONTRACT_ID=YOUR_DEPLOYED_XUSD_CONTRACT_ID
NEXT_PUBLIC_AUCTION_CONTRACT_ID=YOUR_DEPLOYED_AUCTION_CONTRACT_ID
NEXT_PUBLIC_COINGECKO_API_URL=https://api.coingecko.com/api/v3
NEXT_PUBLIC_DISCORD_URL=https://discord.gg/nexlend
NEXT_PUBLIC_TELEGRAM_URL=https://t.me/nexlend
NEXT_PUBLIC_GITHUB_URL=https://github.com/your-org/nexlend
```

### Run Development Server

```bash
pnpm dev
```

Frontend should now be running at `http://localhost:3000`

### Build for Production

```bash
npm run build
npm start
```

---

## 🧪 6. Running Tests

### Contract Tests

```bash
cd contracts
cargo test
```

Run tests for a specific contract:

```bash
cargo test -p nexlend_vault
cargo test -p nexlend_xusd
cargo test -p nexlend_auction
```

### Backend Tests

```bash
cd apps/api
npm test
```

Run with coverage:

```bash
npm run test:coverage
```

### Oracle Tests

```bash
cd oracle
npm test
```

### Frontend Tests

```bash
cd apps/web
npm test
```

Run E2E tests (requires running backend, oracle, and deployed contracts):

```bash
npm run test:e2e
```

### Integration Tests

From project root:

```bash
npm run test:integration
```

---

## 🌍 Network Configuration

### Testnet

- **Network Passphrase:** `Test SDF Network ; September 2015`
- **RPC URL:** `https://soroban-testnet.stellar.org:443`
- **Horizon URL:** `https://horizon-testnet.stellar.org`
- **Friendbot:** `https://friendbot.stellar.org`

### Contract Addresses (Testnet)

- **Vault Contract:** `CXXXXXX...` *(Update after deployment)*
- **xUSD Contract:** `CXXXXXX...` *(Update after deployment)*
- **Auction Contract:** `CXXXXXX...` *(Update after deployment)*

---

## ⚙️ Protocol Parameters

| Parameter | Default Value | Description |
|---|---|---|
| Minimum Collateral Ratio | 150% | Minimum ratio to open a vault |
| Liquidation Threshold | 120% | Ratio below which a vault is liquidatable |
| Dutch Auction Duration | 3600s (1hr) | Time window for auction completion |
| Oracle Feed Interval | 30s | Frequency of price updates pushed on-chain |
| xUSD Mint Fee | 0.5% | Fee charged on each xUSD minting event |
| Liquidation Penalty | 10% | Penalty applied to undercollateralized vaults |

---

## 🔄 Protocol Flow

1. **Deposit Collateral** — User locks a supported on-chain asset (e.g. tokenized real estate token) into a NexLend vault contract.
2. **Mint xUSD** — Based on the current oracle price and minimum collateral ratio, the vault contract calculates the maximum xUSD the user can mint and issues it to their wallet.
3. **Price Monitoring** — The oracle continuously pushes asset prices on-chain. The vault contract tracks each vault's collateralization ratio in real time.
4. **Liquidation Trigger** — If a vault's ratio falls below the liquidation threshold (120%), the vault is flagged as undercollateralized and opened for public liquidation.
5. **Dutch Auction** — The auction contract starts a public Dutch auction for the vault's collateral. The starting price decreases over time, incentivizing liquidators to bid early to secure a discount.
6. **Front-Run Protection** — The contract enforces strict sequencing checks and commit-reveal patterns to prevent liquidators from exploiting mempool visibility to unfairly front-run auction bids.
7. **Debt Settlement** — The winning liquidator repays the vault's outstanding xUSD debt. The collateral is transferred to the liquidator minus the liquidation penalty, which flows to the protocol reserve.
8. **xUSD Burn** — The repaid xUSD is burned by the contract, contracting supply and maintaining the peg.

---

## 🐛 Troubleshooting

### Contract Deployment Fails

**Error:** `insufficient balance`

**Solution:** Fund your account using Friendbot:

```bash
curl "https://friendbot.stellar.org?addr=$(stellar keys address deployer)"
```

### Frontend Can't Connect to Wallet

**Error:** `Failed to connect wallet`

**Solution:**
1. Ensure you have Freighter wallet installed
2. Switch wallet to Testnet network
3. Confirm `NEXT_PUBLIC_STELLAR_NETWORK=testnet` is set in `.env.local`

### Oracle Not Pushing Price Feeds

**Error:** `Stale price data` or `Oracle feed timeout`

**Solution:**
1. Verify `SOROBAN_RPC_URL` is correct in `oracle/.env`
2. Confirm the oracle signer account is funded on testnet
3. Check that `FEED_INTERVAL_MS` is not set too low — Stellar testnet has rate limits
4. Check Stellar testnet status: https://status.stellar.org

### Vault Collateral Ratio Calculation Error

**Error:** `ArithmeticOverflow` or `InvalidCollateralRatio`

**Solution:**
1. Confirm the oracle is actively pushing fresh prices for the collateral asset type
2. Check that the collateral token contract address is registered in the vault's supported asset list
3. Verify `min_collateral_ratio` was correctly set during contract initialization

### Dutch Auction Not Triggering

**Error:** `VaultNotLiquidatable`

**Solution:**
1. Confirm the vault's collateral ratio is genuinely below the liquidation threshold
2. Verify the oracle price feed is current — a stale feed may show an outdated ratio
3. Ensure the auction contract is correctly linked to the vault contract in initialization

### Contract Build Fails

**Error:** `wasm32-unknown-unknown target not found`

**Solution:** Add wasm target:

```bash
rustup target add wasm32-unknown-unknown
```

### Tests Failing

**Error:** `Network connection error`

**Solution:** Ensure all three contracts are deployed and all environment variables are set correctly across `apps/api/.env`, `oracle/.env`, and `apps/web/.env.local`.

---

## 📚 Documentation & Resources

- **Stellar Documentation:** [developers.stellar.org](https://developers.stellar.org/docs/build/smart-contracts)
- **Soroban Docs:** [soroban.stellar.org/docs](https://soroban.stellar.org/docs)
- **Contract Architecture:** [contracts/README.md](./contracts/README.md)
- **Oracle Design:** [oracle/README.md](./oracle/README.md)
- **xUSD Tokenomics:** [docs/xusd.md](./docs/xusd.md)
- **Liquidation & Auction Mechanics:** [docs/liquidation.md](./docs/liquidation.md)
- **Soroban Examples:** [github.com/stellar/soroban-examples](https://github.com/stellar/soroban-examples)

---

## 🤝 Contributing

See our detailed [CONTRIBUTING.md](CONTRIBUTING.md) for coding standards (Rust/Soroban, TypeScript), Git workflow, naming conventions, and the full PR process.

---

## 🗺 Roadmap

### Current Phase (Q2 2026)
- ✅ Vault and collateral management contract
- ✅ xUSD minting and burning logic
- 🚧 Dutch auction engine
- 🚧 Price oracle integration for RWA tokens

### Next Phase (Q3 2026)
- Front-run protection hardening and audit
- Vault health dashboard UI
- Liquidation bot reference implementation
- Multi-collateral vault support
- Mainnet deployment

### Future
- Cross-chain collateral bridging
- Undercollateralized lending via credit scoring
- Governance token and DAO for parameter updates
- xUSD yield strategies
- Advanced analytics and risk dashboards

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- Stellar Development Foundation for the Soroban platform
- MakerDAO and Liquity for pioneering on-chain collateralized debt concepts
- Drips Wave for grants and support
- Open-source contributors and community testers

---

## 📞 Support

Need help? Here's how to get support:

1. Check the [Troubleshooting](#-troubleshooting) section
2. Search [existing issues](https://github.com/your-org/nexlend/issues)
3. Open a [new issue](https://github.com/your-org/nexlend/issues/new) with detailed information
4. Join our [Discord community](https://discord.gg/nexlend) *(if available)*

---

**Built with ❤️ on Stellar**
