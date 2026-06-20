# Fund Me — Crowdfunding Smart Contract

A Solidity crowdfunding contract that accepts ETH contributions with a USD minimum enforced via Chainlink price feeds. Built as part of my blockchain development learning path.

## Overview

This project implements a simple but production-relevant funding flow: users send ETH to the contract, contributions are validated against a real-time USD threshold, and only the contract owner can withdraw the collected funds.

## What Was Built

### `FundMe.sol`
The main contract responsible for receiving and managing funds.

- Accepts ETH through `fund()`, `receive()`, and `fallback()` — any direct transfer triggers the same funding logic
- Enforces a **minimum contribution of $5 USD** using live oracle data
- Tracks each funder's total contribution in a `mapping`
- Maintains a dynamic list of all funders for state cleanup on withdrawal
- Restricts `withdraw()` to the contract owner via a custom `onlyOwner` modifier
- Uses a **custom error** (`NotOwner`) instead of a generic revert string for gas efficiency
- Withdraws the full contract balance using the **`call` pattern** — the recommended approach over `transfer` and `send`

### `PriceConverter.sol`
A reusable Solidity library for ETH → USD conversion.

- Integrates with **Chainlink `AggregatorV3Interface`** to fetch the latest ETH/USD price
- Normalizes oracle decimals (8) to 18-decimal precision for accurate comparisons
- Attached to `uint256` via `using PriceConverter for uint256` in `FundMe`

## Tech Stack

| Technology | Purpose |
|---|---|
| Solidity `0.8.34` | Smart contract language |
| Chainlink Price Feeds | Decentralized ETH/USD oracle on Sepolia |
| SPDX MIT License | Open-source licensing |

## Key Concepts Demonstrated

- **Oracle integration** — reading off-chain price data on-chain via Chainlink
- **Access control** — owner-only withdrawal with a modifier and custom error
- **Library pattern** — separating price logic into a reusable, non-deployable library
- **Receive / fallback functions** — handling plain ETH transfers without calldata
- **Safe fund transfer** — low-level `call` with success check instead of deprecated `transfer`
- **State management** — tracking funders and resetting state after withdrawal

## Contract Architecture

```
User sends ETH
      │
      ▼
receive() / fallback() / fund()
      │
      ▼
PriceConverter.getConversionRate(msg.value)
      │
      ▼
Chainlink ETH/USD Feed (Sepolia)
      │
      ▼
require(value >= $5 USD)
      │
      ▼
Update funder mapping + funders array
      │
      ▼
Owner calls withdraw()
      │
      ▼
Reset state → send full balance via call()
```

## Network & Oracle

| | |
|---|---|
| **Network** | Sepolia testnet |
| **Price Feed** | ETH / USD |
| **Feed Address** | `0x694AA1769357215DE4FAC081bf1f309aDC325306` |

## Project Structure

```
project-fund-me/
├── FundMe.sol          # Main crowdfunding contract
├── PriceConverter.sol  # Chainlink price conversion library
└── README.md
```

## License

MIT
