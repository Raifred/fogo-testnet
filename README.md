# Fogo Testnet CLI — Fast Swaps and Automated DeFi on Solana

[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/Raifred/fogo-testnet/releases)

[![Built for Solana](https://img.shields.io/badge/Platform-Solana-black?logo=solana)](https://solana.com) [![Topics](https://img.shields.io/badge/Topics-auti--swap%20%7C%20automation%20%7C%20bot%20%7C%20fogo-green)](https://github.com/Raifred/fogo-testnet)

![Fogo Banner](https://raw.githubusercontent.com/solana-devs/solana-web3.js/master/docs/static/solana-logo.png)

Table of contents
- About
- Features
- Quick links
- Requirements
- Install
- Configuration
- Basic commands
- Swap examples
- Asset management
- Automation examples
- Debug and logs
- Testing
- Security notes
- Contributing
- License

About
Fogo Terminal is a command-line tool for the Fogo Testnet. It puts common DeFi tasks in your shell. Use it to execute swaps, manage assets, run automated flows, and script incentives. The CLI targets Solana-based token flows and the SVM ecosystem on the Fogo Testnet.

Features
- Swap tokens via on-chain pools and routers.
- Create and manage asset positions.
- Run simple automation jobs and bots.
- Integrate with local scripts and CI pipelines.
- Fast CLI with plain JSON and human output modes.
- Built-in command help and examples.

Quick links
- Releases: https://github.com/Raifred/fogo-testnet/releases
  - Download the release file from that page and execute it on your system.

Requirements
- A Unix-like shell (Linux, macOS, WSL). Windows works via PowerShell or WSL.
- Node.js >= 16 or Go 1.20 if you use source build (refer to the build docs).
- A Solana keypair for testnet. Use solana-keygen or Phantom extension.
- Test SOL in your wallet for fees on the Fogo Testnet.

Install

Option A — Download binary (recommended)
1. Visit the Releases page:
   https://github.com/Raifred/fogo-testnet/releases
2. Download the appropriate binary for your OS from the latest release.
3. Make it executable and move it to your PATH:
   ```bash
   chmod +x fogo
   sudo mv fogo /usr/local/bin/
   ```
4. Verify install:
   ```bash
   fogo --version
   ```

Option B — Source (Node)
1. Clone the repo:
   ```bash
   git clone https://github.com/Raifred/fogo-testnet.git
   cd fogo-testnet
   ```
2. Install deps and build:
   ```bash
   npm install
   npm run build
   npm link
   ```
3. Verify:
   ```bash
   fogo --help
   ```

Configuration
Fogo reads config from:
- Environment variables
- A JSON file at ~/.fogo/config.json
- Command-line flags (highest priority)

Example config (place at ~/.fogo/config.json):
```json
{
  "network": "fogo-testnet",
  "rpc": "https://fogo-rpc.testnet.solana.com",
  "wallet": "~/.config/solana/id.json",
  "output": "human",
  "gasLimit": 500000
}
```

Key environment vars
- FOGO_RPC — RPC endpoint
- FOGO_WALLET — path to keypair
- FOGO_OUTPUT — human | json

Basic commands
Run `fogo --help` for full command list. Key commands:

- fiogo config show
  - Show current config.
- fogo wallet balance
  - Show wallet balance and token list.
- fogo swap quote --from USDC --to FOGO --amount 10
  - Get a quote for a swap.
- fogo swap execute --from USDC --to FOGO --amount 10
  - Execute a swap.
- fogo token list
  - List known tokens on the testnet.
- fogo position create --pool-id <id> --amount 100
  - Create a liquidity position.
- fogo automation run --job <file>
  - Run an automation job script.

Swap examples

Quote a swap
```bash
fogo swap quote \
  --from USDC \
  --to FOGO \
  --amount 100
```
The CLI returns route, estimated price impact, and fee.

Execute a swap
```bash
fogo swap execute \
  --from USDC \
  --to FOGO \
  --amount 100 \
  --slippage 0.5
```
This signs and submits a transaction with your configured wallet. The CLI prints the signature on success.

Atomic swap via router
```bash
fogo swap execute \
  --route routerV2 \
  --from SOL \
  --to USDC \
  --amount 0.5
```

Asset management

Check balances
```bash
fogo wallet balance
```

Deposit into a pool
```bash
fogo position create \
  --pool-id 0xabc123 \
  --amount 50
```

Withdraw
```bash
fogo position withdraw \
  --position-id 42 \
  --amount 25
```

Automation examples
Fogo supports automation scripts in JSON and YAML. Use `fogo automation run` to start a job.

Simple job (buy dip)
file: jobs/buy-dip.json
```json
{
  "name": "buy-dip",
  "triggers": {
    "priceBelow": {
      "pair": "FOGO/USDC",
      "price": 0.95
    }
  },
  "actions": [
    {
      "type": "swap",
      "from": "USDC",
      "to": "FOGO",
      "amount": 50
    }
  ]
}
```
Run it:
```bash
fogo automation run --job jobs/buy-dip.json
```

Bot loop (basic)
Use systemd or a process manager to run `fogo automation run` as a service and let it monitor on-chain data. The CLI outputs structured JSON suitable for logs and alerting.

Debug and logs
- Use `--output json` to stream machine-readable logs.
- Enable debug with `FOGO_DEBUG=1`.
- Logs live in ~/.fogo/logs by default.

Example debug run
```bash
FOGO_DEBUG=1 fogo swap execute --from USDC --to FOGO --amount 10
```

Testing
- Unit tests live in /test.
- Run tests with:
  ```bash
  npm test
  ```
- Integration test harness uses a local validator or a forked testnet. See /test/integration/README.md.

Security notes
- Keep your private keys secure. The CLI reads keypairs from the standard Solana path by default.
- Use the testnet wallet for trials. Do not use mainnet keys on testnet builds.
- Inspect job scripts before running automation jobs. Automations can sign transactions.

Releases and binaries
Download the release file from the Releases page and execute it:
https://github.com/Raifred/fogo-testnet/releases

- The page contains prebuilt binaries and installer archives.
- Pick the file for your OS, download, and run the binary as shown in the Install section.

CI integration
- Use the binary in CI by downloading from the Releases URL.
- Example GitHub Actions snippet:
```yaml
steps:
  - name: Download Fogo
    run: |
      curl -L -o fogo https://github.com/Raifred/fogo-testnet/releases/download/v1.0.0/fogo-linux
      chmod +x fogo
  - name: Run smoke
    run: ./fogo --version
```

Common errors and fixes
- RPC timeout: use a closer RPC or increase `gasLimit`.
- Invalid keypair: ensure `FOGO_WALLET` points to the correct key file and that the key has test SOL.
- Swap failed: check slippage and route quote before execute.

Contributing
- Fork the repo and open pull requests.
- Follow the code style in src/.
- Add tests for new logic.
- Use clear commit messages and link PRs to issues.

Maintainer guide
- Tag releases using semantic versioning.
- Publish binaries in the Releases page.
- Keep changelog updated in CHANGELOG.md.

Roadmap
Planned items:
- Add multi-signer support.
- Add TLS RPC fallback pool.
- Add gas estimation profiles for complex flows.

Credits
- Built with Solana tooling and standard CLI libraries.
- Artwork and icons use public assets and logos.

License
MIT License — see LICENSE file in this repository.

Releases (again)
Download the release file from the Releases page and execute it:
[![Download Releases](https://img.shields.io/badge/Get%20binary-From%20Releases-blue?logo=github)](https://github.com/Raifred/fogo-testnet/releases)

Tags
Topics covered: auti-swap, auto, automation, bot, fogo, fogo-network, fogo-testnet, incentive-testnet, sol, solana, svm, swap, testnet, tools

Contact
Create issues on GitHub or open a discussion in the repo for support.