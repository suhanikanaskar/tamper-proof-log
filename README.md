# Tamper-Proof Project Submission & Evaluation Log
**Group 55 · Ethereum smart contract + IPFS + Node.js + React (MetaMask)**

Students upload a project file. Its **SHA-256 fingerprint** and **IPFS CID** are sealed on an Ethereum smart
contract with a block timestamp. Faculty then log **weighted rubric marks** on-chain. Nothing is ever edited or
deleted: resubmissions and re-evaluations are *appended* to a permanent history.

## Unique features (beyond the basic spec)
| Feature | What it does |
|---|---|
| **Tamper detector** (public) | Drop any file + Group ID. The browser hashes it and the contract says AUTHENTIC / OUTDATED COPY / TAMPERED. No wallet needed. |
| **SHA-256 on-chain** | The file hash is stored next to the CID, so integrity can be proven even without IPFS. Browser and server hashes are cross-checked. |
| **Full version history** | Students may resubmit before the deadline; every version stays on-chain. Submissions lock after evaluation. |
| **Mark modification history** | Re-evaluations are new revisions; old marks stay visible forever. |
| **Weighted rubric** | Report 30 / Code 30 / Presentation 20 / Innovation 20 (admin-editable, must total 100). Weights used are snapshotted per evaluation. |
| **Deadline enforcement** | Admin sets a deadline; the contract rejects late submissions. Live countdown on the dashboard. |
| **Group reservation** | Admin can reserve a Group ID for a specific student wallet; others cannot hijack it. |
| **Integrity certificate** | Printable / save-as-PDF certificate with a record digest, plus shareable verify links (`?verify=55`). |
| **Role-based UI** | Admin / Faculty / Student detected from the wallet automatically. |
| **Zero-config demo mode** | No Pinata keys? Files are stored locally so everything still works. Contract address is auto-wired to the frontend. |

## Requirements
- **Node.js 18 or newer** (https://nodejs.org)
- **MetaMask** browser extension (https://metamask.io)
- Internet for the first `npm run setup` (Hardhat also downloads the Solidity compiler on first compile)

## Quick start (3 commands)
```bash
npm run setup      # installs root + frontend dependencies (once)
npm test           # optional: runs 13 smart-contract tests
npm run dev        # starts blockchain + deploys + seeds demo data + backend + frontend
```
Then open **http://localhost:5173**.

`npm run dev` runs four things together: the local chain (port 8545), the deploy/seed step, the upload server
(port 5000) and the React app (port 5173). Press Ctrl+C to stop everything.
The local chain forgets everything when stopped; `npm run dev` re-deploys fresh each time.

## MetaMask setup (one time)
The app auto-adds/switches to the **Hardhat Local** network (chain ID 31337, RPC `http://127.0.0.1:8545`).
Import these **public Hardhat test accounts** (MetaMask → account menu → *Import account* → paste private key).
Never use them on a real network.

| Role | Address | Private key |
|---|---|---|
| Admin (#0) | `0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266` | `0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80` |
| Faculty (#1) | `0x70997970C51812dc3A010C7d01b50e0d17dc79C8` | `0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d` |
| Student (#2) | `0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC` | `0x5de4111afa1a4b94908f83103eb1f1706367c2e68ca870fc3fb9a804cdab365a` |

**MetaMask nonce errors after restarting the chain:** Settings → Advanced → *Clear activity tab data*.

## Try it (5-minute demo)
1. **Ledger**: Group 55 is pre-seeded with a score. Click it to see its history.
2. Connect as **Student (#2)** → **Submit**: Group `56`, a title, drop a PDF/ZIP → confirm in MetaMask.
3. Switch MetaMask to **Faculty (#1)** → **Evaluate**: pick Group 56, move the sliders, *Log grade on-chain*.
4. **Verify** (no wallet needed): Group 56 + the original file → **AUTHENTIC**. Change the file even slightly → **TAMPERED**.
5. Open the group in the **Ledger** → **Integrity certificate** → Print / Save as PDF.
6. Switch to **Admin (#0)** → set a deadline, change weights, add evaluators.

## Using real IPFS (Pinata), optional
1. Free account at https://app.pinata.cloud → API Keys → create a key and copy the **JWT**.
2. Copy `server/.env.example` to `server/.env` and set `PINATA_JWT=...` (or `PINATA_API_KEY` + `PINATA_SECRET_KEY`).
3. Restart. The server prints `Storage mode: PINATA`. Files open at `https://gateway.pinata.cloud/ipfs/<CID>`.

## Deploying to Sepolia testnet, optional
1. Copy `.env.example` to `.env`; set `SEPOLIA_RPC_URL` (Alchemy/Infura) and `PRIVATE_KEY` of a **test wallet** with Sepolia ETH.
2. `npm run deploy:sepolia` (writes the address into `frontend/src/contract.json`).
3. In `frontend/src/contract.json` set `"chainId": 11155111`; in `frontend/.env` set `VITE_RPC_URL=<your Sepolia RPC>`.
4. Run `npm run server` and `npm run frontend`, with MetaMask on Sepolia.

## Running pieces separately (instead of `npm run dev`)
```bash
npm run chain          # terminal 1
npm run deploy:local   # terminal 2 (then: npm run seed)
npm run server         # terminal 3
npm run frontend       # terminal 4
```

## Project structure
```
contracts/ProjectEvaluationLog.sol   smart contract (roles, versions, rubric, deadline, verify)
scripts/deploy.js                    deploys + writes frontend/src/contract.json (address + ABI)
scripts/seed.js                      demo data (Group 55)
test/ProjectEvaluationLog.test.js    13 automated tests
server/index.js                      upload API (Pinata or local fallback, type + size limits)
frontend/src/                        React + Vite + ethers v6 app (App.jsx, chain.js, styles.css)
```

## Troubleshooting
| Problem | Fix |
|---|---|
| "Contract not deployed yet" | Chain + deploy must run first: use `npm run dev`. |
| "Could not read the chain" | Chain not running, or restarted without redeploying. |
| "Upload server not reachable" | Run `npm run server` (port 5000). |
| Port already in use | Close old terminals, or change `PORT` in `server/.env`. |
| Revert: "Already evaluated" | By design: evaluated projects are locked. |

## Security notes
- Never put a funded wallet's private key in `.env`; use a test wallet. `.env` files are git-ignored.
- On a public chain anyone can claim an unused Group ID; use *Reserve a group* (Admin) to bind IDs to student wallets.
