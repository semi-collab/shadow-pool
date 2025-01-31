# ShadowPool

A privacy-preserving layer for Bitcoin transactions on Stacks L2, enabling confidential deposits and withdrawals using zero-knowledge proofs and Merkle trees.

## Overview

ShadowPool is a smart contract that implements a privacy layer for Bitcoin transactions on the Stacks L2 network. It allows users to:

- Make confidential deposits of tokens
- Withdraw tokens privately without revealing transaction history
- Maintain transaction privacy using zero-knowledge proofs
- Verify transaction validity without exposing sensitive details

## Technical Architecture

### Core Components

1. **Merkle Tree**

   - Height: 20 levels
   - Maximum capacity: 2^20 deposits
   - Zero value: `0x0000000000000000000000000000000000000000000000000000000000000000`

2. **Data Structures**
   - `deposits`: Maps commitments to deposit information (leaf index and timestamp)
   - `nullifiers`: Tracks used nullifiers to prevent double-spending
   - `merkle-tree`: Stores the hash values at each level and index

### Key Features

#### SIP-010 Token Standard Compliance

The contract implements the SIP-010 fungible token trait, ensuring compatibility with standard token operations:

- Transfer functionality
- Balance queries
- Supply management
- Token metadata (name, symbol, decimals)

#### Privacy Mechanisms

1. **Deposit Process**

   - Users create commitments for deposits
   - Commitments are inserted into a Merkle tree
   - Deposit amounts are hidden within commitments

2. **Withdrawal Process**
   - Uses zero-knowledge proofs for verification
   - Implements nullifier tracking to prevent double-spending
   - Allows private transfers to recipient addresses

## Smart Contract Interface

### Public Functions

#### `deposit`

```clarity
(deposit
    (commitment (buff 32))
    (amount uint)
    (token <ft-trait>))
```

- Creates a new deposit in the privacy pool
- Parameters:
  - `commitment`: 32-byte commitment hash
  - `amount`: Token amount to deposit
  - `token`: SIP-010 token contract reference
- Returns: Leaf index of the deposit in the Merkle tree

#### `withdraw`

```clarity
(withdraw
    (nullifier (buff 32))
    (root (buff 32))
    (proof (list 20 (buff 32)))
    (recipient principal)
    (token <ft-trait>)
    (amount uint))
```

- Processes a private withdrawal
- Parameters:
  - `nullifier`: Unique nullifier to prevent double-spending
  - `root`: Merkle root for proof verification
  - `proof`: Merkle proof (20 elements)
  - `recipient`: Withdrawal recipient address
  - `token`: SIP-010 token contract reference
  - `amount`: Token amount to withdraw

### Read-Only Functions

#### `get-current-root`

- Returns the current Merkle tree root

#### `is-nullifier-used`

- Checks if a nullifier has been previously used

#### `get-deposit-info`

- Retrieves information about a specific deposit

## Error Codes

| Code | Description              |
| ---- | ------------------------ |
| 1001 | Not authorized           |
| 1002 | Invalid amount           |
| 1003 | Insufficient balance     |
| 1004 | Invalid commitment       |
| 1005 | Nullifier already exists |
| 1006 | Invalid proof            |
| 1007 | Merkle tree full         |

## Security Considerations

1. **Zero-Knowledge Proofs**

   - All withdrawals require valid zero-knowledge proofs
   - Proofs verify ownership without revealing deposit details

2. **Double-Spending Prevention**

   - Nullifier tracking prevents multiple withdrawals of the same deposit
   - Each nullifier can only be used once

3. **Merkle Tree Security**
   - Implements secure hash combining for tree updates
   - Validates all hash operations
   - Prevents invalid tree states

## Implementation Notes

1. **Merkle Tree Management**

   - Efficient parent node updates
   - Automatic tree height management
   - Zero value handling for empty nodes

2. **Token Handling**
   - Safe token transfers using SIP-010 standard
   - Contract-controlled custody during deposits
   - Direct transfers to recipients during withdrawals

## Usage Example

1. **Making a Deposit**

```clarity
;; Generate a commitment off-chain
;; Call deposit function with commitment
(contract-call? .shadowpool deposit
    commitment
    amount
    token-contract)
```

2. **Processing a Withdrawal**

```clarity
;; Generate proof off-chain
;; Call withdraw function
(contract-call? .shadowpool withdraw
    nullifier
    root
    proof
    recipient
    token-contract
    amount)
```

## Best Practices

1. **Deposit Security**

   - Store commitment data securely off-chain
   - Never reuse commitments
   - Verify deposit inclusion before withdrawal

2. **Withdrawal Security**
   - Generate unique nullifiers for each withdrawal
   - Verify proof validity before submission
   - Keep private keys secure

## Development and Testing

To interact with the contract:

1. Deploy the contract to Stacks testnet/mainnet
2. Initialize with supported token contracts
3. Test deposit and withdrawal flows
4. Monitor transaction privacy and security

## Limitations

1. Fixed Merkle tree height (20 levels)
2. Maximum deposit capacity of 2^20 entries
3. Requires off-chain proof generation
4. Token-specific implementation

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request with detailed description
4. Ensure all tests pass
