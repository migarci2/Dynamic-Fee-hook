# Dynamic Fee Hook for Uniswap V4

A Uniswap V4 hook that dynamically adjusts swap fees based on network gas prices, implementing a moving average algorithm to optimize fees during different network conditions.

## Overview

The `GasPriceFeesHook` is a smart contract hook for Uniswap V4 that automatically adjusts liquidity provider (LP) fees based on current gas prices relative to a moving average. This creates a more efficient market by:

- **Reducing fees during high gas prices** - Makes swaps more attractive when network is congested
- **Increasing fees during low gas prices** - Captures more value for LPs during favorable network conditions
- **Maintaining balanced fees** - Uses BASE_FEE when gas prices are stable

## Features

- **Dynamic Fee Adjustment**: Automatically adjusts fees based on real-time gas price data
- **Moving Average Tracking**: Maintains a cumulative moving average of gas prices across all swaps
- **Three-Tier Fee Structure**:
  - 50% discount (0.25%) when gas price > 110% of moving average
  - Base fee (0.5%) when gas price is within ±10% of moving average
  - 100% premium (1.0%) when gas price < 90% of moving average

## How It Works

### Fee Calculation Algorithm

1. **Moving Average Update**: After each swap, the contract updates the moving average:
   ```
   New Average = ((Old Average × Transaction Count) + Current Gas Price) / (Transaction Count + 1)
   ```

2. **Fee Determination**: Before each swap, the contract compares current gas price to the moving average:
   - If `gasPrice > movingAverage × 1.1` → Fee = 0.25% (BASE_FEE / 2)
   - If `gasPrice < movingAverage × 0.9` → Fee = 1.0% (BASE_FEE × 2)
   - Otherwise → Fee = 0.5% (BASE_FEE)

### Hook Implementation

The hook implements the following Uniswap V4 hook functions:

- `beforeInitialize`: Validates that the pool uses dynamic fees
- `beforeSwap`: Calculates and returns the appropriate fee based on current gas price
- `afterSwap`: Updates the moving average gas price tracker

## Technical Details

### Smart Contract Architecture

```solidity
contract GasPriceFeesHook is BaseHook {
    uint128 public movingAverageGasPrice;      // Current moving average
    uint104 public movingAverageGasPriceCount;  // Number of tracked transactions
    uint24 public constant BASE_FEE = 5000;     // 0.5% base fee (in basis points)
}
```

### Hook Permissions

The contract requires the following hook permissions:
- `beforeInitialize`: To ensure the pool uses dynamic fees
- `beforeSwap`: To calculate and set the fee for each swap
- `afterSwap`: To update the moving average gas price

## Installation

### Prerequisites

- [Foundry](https://book.getfoundry.sh/getting-started/installation)
- Git

### Setup

1. Clone the repository:
```bash
git clone https://github.com/migarci2/Dynamic-Fee-hook.git
cd Dynamic-Fee-hook
```

2. Install dependencies:
```bash
forge install
```

3. Build the project:
```bash
forge build
```

## Usage

### Running Tests

Execute the full test suite:
```bash
forge test
```

Run tests with detailed output:
```bash
forge test -vvv
```

Run specific tests:
```bash
forge test --match-test test_feeUpdatesWithGasPrice
```

### Gas Snapshots

Generate gas usage snapshots:
```bash
forge snapshot
```

### Deployment

Deploy the hook to a network:
```bash
forge script script/DeployHook.s.sol --rpc-url <your_rpc_url> --private-key <your_private_key>
```

**Note**: The hook must be deployed at an address that matches the required hook flags. Use Uniswap V4's CREATE2 mining strategy to find a valid address.

### Code Formatting

Format Solidity code:
```bash
forge fmt
```

## Project Structure

```
Dynamic-Fee-hook/
├── src/
│   └── GasPriceFeesHook.sol      # Main hook implementation
├── test/
│   └── GasPriceFeeHook.t.sol     # Test suite
├── script/                        # Deployment scripts
├── lib/                          # Dependencies
│   ├── forge-std/               # Foundry standard library
│   └── v4-periphery/            # Uniswap V4 periphery contracts
├── foundry.toml                  # Foundry configuration
└── README.md                     # This file
```

## Testing

The test suite (`GasPriceFeeHook.t.sol`) includes comprehensive tests for:

- Fee calculation at different gas price levels
- Moving average updates after swaps
- Pool initialization with dynamic fees
- Multiple swap scenarios with varying gas prices

### Example Test Scenarios

1. **Base Fee Test**: Swap at current moving average gas price
2. **High Gas Price Test**: Swap when gas is 110%+ of average (reduced fees)
3. **Low Gas Price Test**: Swap when gas is 90%- of average (increased fees)
4. **Moving Average Update**: Verification that average updates correctly after each swap

## Configuration

### Solidity Compiler Settings

- Solidity Version: `0.8.26`
- EVM Version: `cancun`
- Optimizer Runs: `800`
- Via IR: `false`

These settings are configured in `foundry.toml`.

## Dependencies

- [Uniswap V4 Core](https://github.com/Uniswap/v4-core)
- [Uniswap V4 Periphery](https://github.com/Uniswap/v4-periphery)
- [Forge Standard Library](https://github.com/foundry-rs/forge-std)

## Security Considerations

- The hook uses `tx.gasprice` which can be manipulated by miners/validators
- Moving average is calculated on-chain, starting from deployment gas price
- The fee override flag (`OVERRIDE_FEE_FLAG`) is set for per-swap fee adjustment
- Pools using this hook must be initialized with the `DYNAMIC_FEE_FLAG`

## License

MIT License - see [LICENSE](LICENSE) file for details

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Resources

- [Uniswap V4 Documentation](https://docs.uniswap.org/contracts/v4/overview)
- [Foundry Book](https://book.getfoundry.sh/)
- [Uniswap V4 Hooks Guide](https://docs.uniswap.org/contracts/v4/concepts/hooks)

## Author

Created by [@migarci2](https://github.com/migarci2)

## Acknowledgments

- Uniswap Labs for V4 architecture and periphery contracts
- Foundry team for the development toolkit
