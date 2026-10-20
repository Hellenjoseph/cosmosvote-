# First-Time Contributor Onboarding

Welcome to CosmosVote! This guide takes you from a fresh checkout to running
the project's checks and making a first contribution. You do not need prior
Stellar or Soroban experience to run the tests or work on most issues.

## Prerequisites

Required for contract development and the standard checks:

| Tool | Version | Purpose |
| --- | --- | --- |
| Git | Recent version | Clone the repository and work with branches |
| Rust and Cargo | Rust 1.78.0 (the version used by CI) | Build and test the contracts |
| `wasm32-unknown-unknown` target | Rust target | Build deployable contracts |
| GNU Make | Any recent version | Run the repository's common commands |

Node.js 20 and npm are needed for frontend or backend work. Docker with the
Docker Compose plugin is optional; it is used to run a local Stellar node.
The Stellar CLI is optional unless you want to deploy and invoke contracts
manually.

Install Rust and the WebAssembly target:

```bash
rustup toolchain install 1.78.0
rustup default 1.78.0
rustup target add wasm32-unknown-unknown
```

Install Node.js 20 using your preferred version manager, then verify the tools:

```bash
rustc --version
cargo --version
rustup target list --installed
make --version
node --version
npm --version
```

## Get the code and set up your environment

Fork `mercyJJ/cosmosvote` on GitHub, then clone your fork. Add the upstream
repository so you can update your local `main` branch:

```bash
git clone https://github.com/<your-username>/cosmosvote.git
cd cosmosvote
git remote add upstream https://github.com/mercyJJ/cosmosvote.git
git remote -v
```

The contract tests run in a local Soroban test environment and do not require a
wallet, RPC endpoint, deployed contract, secret key, or Docker. Do not put a
real Stellar secret key in a tracked file. If you later need the optional local
network, see [Local Soroban Contract Deployment](./local-deployment.md).

To work on the frontend or backend, install its locked dependencies:

```bash
cd frontend && npm ci
cd ../backend && npm ci
cd ..
```

## Run your first tests

From the repository root, run the Rust unit and property tests:

```bash
make test
```

You can run a smaller package-specific test suite while iterating:

```bash
cargo test -p cosmosvote-governance --features testutils
cargo test -p cosmosvote-token --features testutils
```

For frontend or backend changes, also run the relevant JavaScript tests:

```bash
cd frontend && npm test -- --run
cd ../backend && npm test -- --runInBand
```

The first Cargo run may take several minutes while dependencies compile. A
successful run ends with a test summary and exit status `0`.

## Run your first lint and build checks

Run the Rust linter and formatting check from the repository root:

```bash
make lint
make fmt-check
```

For frontend changes, run:

```bash
cd frontend
npm run lint
npm run type-check
```

For backend changes, run:

```bash
cd backend
npm run build
```

To confirm contracts compile to WebAssembly, run `make build` from the
repository root. Run checks relevant to your change before opening a PR; CI
will run the project checks again.

## Find a good first issue

Browse [open issues labeled `good first issue`](https://github.com/mercyJJ/cosmosvote/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22).
If there are no matching issues, look for open issues labeled `documentation`
or `help wanted`. Read the issue and any linked design notes, then leave a
comment asking to work on it before starting. Maintainers can confirm that the
issue is still available and clarify its scope.

For the branch, commit, and pull request conventions, follow
[CONTRIBUTING.md](../CONTRIBUTING.md).

## Create a proposal end-to-end

This optional exercise uses a local network and the Stellar CLI. It does not
send transactions to testnet or mainnet. You need Docker Compose and the
Stellar CLI installed; local contract calls do not require Stellar expertise.

1. Start the local Soroban node and wait for it to become healthy:

   ```bash
   docker compose up -d stellar-node
   docker compose ps
   ```

2. Create and fund a local CLI identity:

   ```bash
   stellar keys generate --global alice --network local
   stellar keys fund alice --network local
   stellar keys address alice
   ```

3. Build and deploy the token and governance contracts to the local network.
   Follow the local deployment walkthrough, which includes the required
   contract initialization steps:
   [Local Soroban Contract Deployment](./local-deployment.md).

4. Set the contract IDs printed by deployment in your shell. Replace the
   placeholders with the IDs for the local network:

   ```bash
   export TOKEN_CONTRACT_ID=<local-token-contract-id>
   export GOVERNANCE_CONTRACT_ID=<local-governance-contract-id>
   ```

5. Create a proposal. The proposer must have the minimum token balance
   configured on the governance contract. `quorum` must be positive and no
   larger than the token supply; `duration` must be between 60 and 2,592,000
   seconds:

   ```bash
   stellar contract invoke \
     --id "$GOVERNANCE_CONTRACT_ID" \
     --source alice \
     --network local \
     -- create_proposal \
     --proposer alice \
     --title "My first proposal" \
     --description "Document a small improvement for the community." \
     --quorum 1000 \
     --duration 3600 \
     --link 'null' \
     --treasury_action 'null'
   ```

6. Copy the proposal ID from the invocation result. The contract records a
   `proposal_created` event; event data and the proposal state can be inspected
   with the tools described in [Contract State Inspection](./contract-state-inspection.md).
   Stop the local node when finished:

   ```bash
   docker compose down
   ```

This creates a proposal only; it does not execute or approve it. Contract
deployment and invocation commands can vary between Stellar CLI releases. If
the local CLI rejects an argument, check `stellar contract invoke --help` and
the contract's `create_proposal` definition before proceeding.

## Troubleshooting

| Error or symptom | What to try |
| --- | --- |
| `rustup: command not found` | Install Rust with [rustup](https://rustup.rs), then open a new terminal so its environment is loaded. |
| `can't find crate for \`core\`` or target unavailable | Run `rustup target add wasm32-unknown-unknown` using the active toolchain. |
| Cargo cannot download a dependency | Check network/proxy access, then retry `cargo test`; use `cargo test --locked` if the lockfile was changed unexpectedly. |
| `make: command not found` | Install GNU Make, or run the corresponding Cargo command directly (for example, `cargo test --all --features testutils`). |
| `npm ci` fails or reports a lockfile mismatch | Confirm Node.js 20 is active and that `package-lock.json` is committed; do not replace `npm ci` with an uncommitted lockfile update. |
| Frontend reports a missing/invalid contract ID | Frontend contract addresses are deployment configuration. Use a matching network deployment and check `frontend/.env.example` and `docs/environment-variables.md`. |
| Docker reports a port conflict or the node never becomes healthy | Check whether port 8001 is in use with `docker compose ps`, then inspect `docker compose logs stellar-node`. |
| `stellar keys fund` or invocation fails | Confirm `--network local` is selected and the local node is healthy; do not use a testnet or mainnet identity for this exercise. |

If a problem remains, include the exact command, full error output, operating
system, and relevant tool versions when asking in
[GitHub Discussions](https://github.com/mercyJJ/cosmosvote/discussions).
