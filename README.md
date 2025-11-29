# Encrypted Certification Filter — Zama FHEVM dApp

> Privacy-preserving aggregation of qualification levels.
> Users submit **encrypted levels**, the contract aggregates them on-chain, but never sees any plaintext.

![Built with FHEVM](https://img.shields.io/badge/Built%20with-Zama%20FHEVM-00d9ff?logo=ethereum\&logoColor=white)
![Network](https://img.shields.io/badge/Network-Sepolia-blue)
![License](https://img.shields.io/badge/License-MIT-green)

---

## Overview

**Encrypted Certification Filter** is a minimal Zama FHEVM demo that lets you:

* create a **certificate bucket** identified by `certId`;
* let many users submit their **qualification level** as an **encrypted `uint16`**;
* maintain an **encrypted running total** of all submissions;
* optionally make this total **publicly decryptable**, without revealing any individual submission.

Typical use cases:

* private skill / qualification scoring;
* privacy-preserving certification systems;
* “filter by minimum level” checks where raw scores must stay hidden.

---

## How It Works

### Data model

The contract stores per-certificate aggregate state:

```solidity
struct CertificateData {
    bool exists;
    euint16 totalLevel; // encrypted sum of all submitted levels
    uint256 submissions; // number of submissions
}
```

Indexed by:

```solidity
mapping(bytes32 => CertificateData) private certs;
```

Where:

* `certId : bytes32` — arbitrary identifier for a certificate pool;
* `totalLevel : euint16` — encrypted sum of all submitted levels for this certificate;
* `submissions : uint256` — how many submissions were received.

### Flow

1. **Initialize certificate**

   ```solidity
   function initCertificate(bytes32 certId) public
   ```

   * Creates a new certificate bucket.
   * Sets `totalLevel = 0` (as encrypted `euint16`).
   * Marks the certificate as existing and allows the contract to reuse `totalLevel` internally:

   ```solidity
   C.totalLevel = FHE.asEuint16(0);
   FHE.allowThis(C.totalLevel);
   ```

2. **Submit encrypted level**

   Frontend:

   * Uses Zama **Relayer SDK** to encrypt a `uint16` level:

     ```ts
     const input = relayer.createEncryptedInput(CONTRACT_ADDRESS, userAddress);
     input.add16(BigInt(level));
     const { handles, inputProof } = await input.encrypt();
     ```

   * Concatenates `handle (32 bytes) + proof (rest)` into a single `bytes` blob and passes it to:

     ```solidity
     submitCertificate(certId, encryptedData);
     ```

   Contract:

   ```solidity
   // Split: first 32 bytes = handle, rest = proof
   bytes memory handle = encryptedData[:32];
   bytes memory proof = encryptedData[32:];

   euint16 lvl = FHE.fromExternal(
       externalEuint16.wrap(bytes32(handle)),
       proof
   );

   euint16 newTotal = FHE.add(C.totalLevel, lvl);
   C.totalLevel = newTotal;
   FHE.allowThis(C.totalLevel); // keep contract access
   C.submissions++;
   ```

   The contract:

   * **never sees the plaintext level;**
   * only works with encrypted values and homomorphic addition.

3. **Make result public (optional)**

   ```solidity
   function makePublic(bytes32 certId) external
   ```

   * Calls `FHE.makePubliclyDecryptable(C.totalLevel)`.
   * After that, **anyone** can use `publicDecrypt` with the handle to reveal `totalLevel` (but still not the individual contributions).

4. **Read encrypted handle & number of submissions**

   ```solidity
   function certificateHandle(bytes32 certId) external view returns (bytes32);

   function submissions(bytes32 certId) external view returns (uint256);
   ```

   * `certificateHandle` returns a `bytes32` handle for `totalLevel`.
   * `submissions` returns how many encrypted levels have been submitted for that certificate.

---

## Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.27;

import { FHE, euint16, externalEuint16 } from "@fhevm/solidity/lib/FHE.sol";
import { ZamaEthereumConfig } from "@fhevm/solidity/config/ZamaConfig.sol";

contract EncryptedCertificationFilter is ZamaEthereumConfig {
    struct CertificateData {
        bool exists;
        euint16 totalLevel;
        uint256 submissions;
    }

    mapping(bytes32 => CertificateData) private certs;

    event CertificateSubmitted(bytes32 indexed certId, uint256 newCount);
    event MadePublic(bytes32 indexed certId);

    function initCertificate(bytes32 certId) public { ... }

    function submitCertificate(bytes32 certId, bytes calldata encryptedData) external { ... }

    function makePublic(bytes32 certId) external { ... }

    function certificateHandle(bytes32 certId) external view returns (bytes32) { ... }

    function submissions(bytes32 certId) external view returns (uint256) { ... }
}
```

Key points:

* Uses only official Zama Solidity library:

  ```solidity
  import { FHE, euint16, externalEuint16 } from "@fhevm/solidity/lib/FHE.sol";
  import { ZamaEthereumConfig } from "@fhevm/solidity/config/ZamaConfig.sol";
  ```

* Uses FHE arithmetic (`FHE.add`) on `euint16` (safe for small levels).

* No FHE operations in `view` functions (only `FHE.toBytes32`).

---

## Frontend

The frontend is a single HTML file, using:

* **Vanilla JS** + **ethers v6** (via CDN).
* **Zama Relayer SDK** browser bundle (`relayer-sdk-js`).
* Minimal, responsive UI with 3 main cards: **Initialize**, **Submit**, **View**.

### Network & SDK config

```js
const CONFIG = {
  RELAYER_URL: "https://relayer.testnet.zama.org",
  GATEWAY_URL: "https://gateway.testnet.zama.org",
  CONTRACT_ADDRESS: "0x5b30beD0BA9D796f1e58FA36130e513E3EBEEEe7"
};
```

Relayer is initialized with `SepoliaConfig`:

```js
await initSDK();
relayer = await createInstance({
  ...SepoliaConfig,
  relayerUrl: CONFIG.RELAYER_URL,
  gatewayUrl: CONFIG.GATEWAY_URL,
  network: window.ethereum
});
```

### UI Sections

#### 1. Initialize

* **Certificate ID** (numeric input)
* **Random** button – generates a random ID and fills all inputs.
* **Initialize** button:

  * Calls `initCertificate(certId)` on the contract (after converting number → `bytes32`).
  * Shows status message (success / error).

#### 2. Submit

* **Certificate ID** (numeric)
* **Qualification Level** (0–65535)
* **Encrypt & Submit** button:

  1. Uses Relayer SDK:

     ```js
     const input = relayer.createEncryptedInput(CONTRACT_ADDRESS, address);
     input.add16(BigInt(level));
     const { handles, inputProof } = await input.encrypt();
     ```

  2. Concatenates `handle` + `inputProof` into `encryptedData`.

  3. Sends `submitCertificate(certId, encryptedData)` transaction.

  4. Shows live debug log in the **debug-log** panel.

#### 3. View

* **Certificate ID** input.
* **Get Submissions** button:

  * Calls `submissions(certId)` and prints:

    ```text
    Submissions: <count>
    ```

> Note: public decryption of the aggregated `totalLevel` is done off-chain using the handle returned by `certificateHandle(certId)` and the Zama Relayer SDK (`publicDecrypt`).

---

## Project Structure

```text
.
├── contracts/
│   └── EncryptedCertificationFilter.sol   # FHEVM smart contract
├── frontend/
│   └── index.html                         # single-page frontend + logic
├── README.md                              # this file
└── ...                                    # scripts, configs, etc.
```

---

## Running Locally

1. Clone the repo:

   ```bash
   git clone https://github.com/<your-name>/<your-repo>.git
   cd <your-repo>/frontend
   ```

2. Serve the static HTML with any HTTP server, e.g.:

   **Option 1: `serve`**

   ```bash
   npm install -g serve
   serve .
   ```

   **Option 2: `http-server`**

   ```bash
   npm install -g http-server
   http-server .
   ```

3. Open the printed URL in your browser.

4. Connect MetaMask (or another EVM wallet) and switch to **Sepolia** (or the network where you deployed the contract).

5. Ensure `CONFIG.CONTRACT_ADDRESS` in the script matches your deployed contract address.

---

## Future Ideas

* Track **avera
