# Kyber-Lite Aggregator

A minimalist, experimental Decentralized Exchange (DEX) liquidity aggregator designed to demonstrate core principles and technical architecture inspired by Kyber Network's strategic direction. This project focuses on a simplified model: single-hop swaps, single liquidity source (Uniswap V3), and clear separation of on-chain execution and off-chain computation.

---

## 🚀 Project Goals

The primary goal of this experimental project, "Kyber-Lite Aggregator," is to build a functional prototype DApp capable of executing a secure and efficient swap transaction. It aims to clarify the core architectural and technical principles by:

-   **Single Integration:** Interacting solely with Uniswap V3, a well-audited and deeply liquid protocol.
-   **Single-Hop Swaps:** Executing direct swaps (e.g., WETH -> USDC), excluding complex multi-hop routing algorithms.
-   **Logic Separation:** Clearly separating the transaction execution layer (on-chain) from the computation and quoting layer (off-chain) to optimize gas costs and performance.

Through this project, we aim to provide a practical analysis of the trade-offs between cost, security, and user experience in the design of a DEX aggregator.

---

## 🛠️ Technologies Used

### On-chain (Solidity Smart Contract - Backend)

-   **Language:** [Solidity](https://soliditylang.org/) ^0.8.20
-   **Development Environment:** [Hardhat](https://hardhat.org/) - A comprehensive Ethereum development environment for compiling, testing, and deploying smart contracts.
-   **Security Libraries:** [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/5.x/) - Utilizing audited modules like `ReentrancyGuard` (for reentrancy attack prevention) and `SafeERC20` (for safe ERC20 token interactions).
-   **Uniswap V3:** Integration with Uniswap V3's `ISwapRouter` and `IQuoterV2` interfaces for swapping and quoting.

### Off-chain (Node.js Script - Interaction Logic)

-   **Platform:** [Node.js](https://nodejs.org/en)
-   **Blockchain Interaction Library:** [Ethers.js](https://docs.ethers.org/v6/) - A complete library for interacting with Ethereum, including contract object creation, transaction sending, and complex data type handling.
-   **Uniswap SDKs:** `@uniswap/v3-sdk`, `@uniswap/sdk-core`, `@uniswap/smart-order-router` (though `AlphaRouter` is not fully utilized for complex routing in this simplified version, core SDK components are used for quoting).

---

## 🏗️ System Architecture and Development Process

The development process is structured systematically, adhering to best practices in smart contract and DApp development. The system architecture follows a hybrid model, reasonably distributing tasks between on-chain and off-chain environments.

### Phase 1: On-chain Execution Layer (Backend) Design and Deployment

This phase focuses on building `KyberLiteAggregator.sol`, a smart contract acting as the final transaction execution layer, ensuring atomicity and security.

#### Key Design Decisions:

1.  **Interface and Data Structure Design:** Directly uses `ExactInputSingleParams` defined by Uniswap V3's `ISwapRouter` interface for compatibility and scalability.
2.  **Business Logic Programming:** The core `swap()` function acts as a trusted proxy. It receives pre-prepared parameters from off-chain, pulls input tokens, approves the Uniswap `SwapRouter`, calls `exactInputSingle`, and sends output tokens directly to the user to optimize gas costs.
3.  **Security Layer Integration:**
    * **Reentrancy Protection:** The entire `swap()` function is protected by the `nonReentrant` modifier from OpenZeppelin's `ReentrancyGuard`.
    * **Slippage Protection:** The contract enforces a user-provided safety limit via the `amountOutMinimum` parameter, ensuring transactions revert if the actual output falls below the specified minimum.

### Phase 2: Off-chain Computation Layer (Interaction Logic) Design and Development

This phase focuses on building an off-chain script using Node.js. This script acts as the "brain" of the aggregator, performing heavy computations and preparing transaction data optimally before requesting user signature.

#### Key Processes:

1.  **Quoting Phase:** The script performs a read-only call to Uniswap V3's `QuoterV2` (`quoteExactInputSingle`) to provide an accurate quote. This `staticCall` is gas-free and real-time, allowing the UI to display expected rates without requiring user wallet interaction.
2.  **Transaction Construction Phase:** Based on the quote and a user-configured slippage tolerance (e.g., 0.5%), the script calculates `amountOutMinimum` and constructs a complete transaction object (including token addresses, amounts, pool fee, and `amountOutMinimum`).
3.  **Execution Phase:** The script uses Ethers.js to connect to the user's wallet provider (e.g., MetaMask). It then prompts the user to sign and send the constructed transaction object to call the `swap()` function on the deployed `KyberLiteAggregator.sol` contract.

---

## 🚀 Getting Started

### Prerequisites

-   Node.js (LTS version recommended)
-   npm or yarn
-   Git
-   A `.env` file with your private key and Infura/Alchemy URL for a Goerli (or desired) network.
-   Deployed `KyberLiteAggregator.sol` contract address.

### Installation

1.  **Clone the repository:**
    ```bash
    git clone [https://github.com/your-username/Kyber-Lite-Aggregator.git](https://github.com/your-username/Kyber-Lite-Aggregator.git)
    cd Kyber-Lite-Aggregator
    ```

2.  **Install dependencies:**
    ```bash
    npm install
    # or
    yarn install
    ```

3.  **Configure Environment Variables:**
    Create a `.env` file in the root directory based on `.env.example`:

    ```
    PRIVATE_KEY="your_ethereum_wallet_private_key_here"
    INFURA_URL_GOERLI="[https://goerli.infura.io/v3/YOUR_INFURA_PROJECT_ID](https://goerli.infura.io/v3/YOUR_INFURA_PROJECT_ID)"
    ETHERSCAN_API_KEY="YOUR_ETHERSCAN_API_KEY" # Optional, for contract verification

    # After deployment, update this with your deployed contract address
    KYBERLITE_AGGREGATOR_ADDRESS="0xYourDeployedKyberLiteAggregatorAddress"
    ```
    Replace placeholder values with your actual keys and addresses.

### Deployment (Smart Contract)

To deploy `KyberLiteAggregator.sol` to the Goerli testnet (or another configured network):

```bash
npx hardhat run scripts/deploy.js --network goerli
