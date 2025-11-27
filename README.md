❤️ EncryptedCertificationFilter — Private Match (FHEVM dApp)

A decentralized fully homomorphic encrypted matchmaking dApp on Ethereum (Sepolia testnet) using Zama’s FHEVM protocol.
Profiles and preferences are encrypted → matched on-chain → only the final match result is decryptable.
No data leaks. No exposure of personal information.

⚡ Features

Publish encrypted user profiles (age, gender, interests, region)

Submit encrypted match preferences

Homomorphic computation of match compatibility directly on-chain


Zero knowledge of inputs — full privacy preserved

Modern dual-column glassmorphic UI built with pure HTML + CSS

Powered by Zama Relayer SDK v0.3.0 and Ethers.js v6

🛠 Quick Start
Prerequisites

Node.js ≥ 20

npm / yarn / pnpm

MetaMask or any injected Ethereum-compatible wallet

Installation
Clone the repository
git clone <your-repo-url>
cd EncryptedCertificationFilter

Install dependencies
npm install

Set up environment variables
npx hardhat vars set MNEMONIC
npx hardhat vars set INFURA_API_KEY
npx hardhat vars set ETHERSCAN_API_KEY   # optional

Compile Contracts
npm run compile

Run Tests
npm run test

Deploy to Local Network
npx hardhat node
npx hardhat deploy --network localhost

Deploy to Sepolia FHEVM Testnet
npx hardhat deploy --network sepolia
npx hardhat verify --network sepolia 

CONTRACT_ADDRESS: "0x5b30beD0BA9D796f1e58FA36130e513E3EBEEEe7"


📁 Project Structure
tinderdao-private-match/
├── contracts/
│   └── EncryptedCertificationFilter.sol     # Main FHE-enabled matchmaking contract
├── deploy/                                  # Deployment scripts
├── frontend/                                # Web UI (FHE Relayer integration)
│   └── index.html
├── hardhat.config.js                        # Hardhat + FHEVM config
└── package.json                             # Dependencies and npm scripts

📜 Available Scripts
Command	Description
npm run compile	Compile all smart contracts
npm run test	Run unit tests
npm run clean	Clean build artifacts
npm run start	Launch frontend locally
npx hardhat deploy --network sepolia	Deploy to FHEVM Sepolia testnet
npx hardhat verify	Verify contract on Etherscan
🔗 Frontend Integration

The frontend (pure HTML + vanilla JS) uses:

@zama-fhe/relayer-sdk v0.3.0

ethers.js v6.13

Web3 wallet (MetaMask) connection

Workflow:

Connect wallet

Encrypt & Submit a preference query (desired criteria)

Compute match handle via computeMatchHandle()

Make public the result using makeMatchPublic()

Publicly decrypt → get final result (MATCH ✅ / NO MATCH ❌)

🧩 FHEVM Highlights

Encrypted types: euint8, euint16

Homomorphic operations: FHE.eq, FHE.and, FHE.or, FHE.gt, FHE.lt

Secure access control using FHE.allow & FHE.allowThis

Public decryption enabled with FHE.makePubliclyDecryptable

Frontend encryption/decryption handled via Relayer SDK proofs

📚 Documentation

Zama FHEVM Overview

Relayer SDK Guide

Solidity Library: FHE.sol

Ethers.js v6 Documentation

🆘 Support

🐛 GitHub Issues: Report bugs or feature requests

💬 Zama Discord: discord.gg/zama-ai
 — community help

📄 License

BSD-3-Clause-Clear License
See the LICENSE
 file for full details.