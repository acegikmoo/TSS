# Solana TSS API Backend

A Rust-based REST API backend implementing Threshold Signature Scheme (TSS) for Solana transactions using the MuSig2 multi-party signature protocol.

## Overview

This project provides HTTP endpoints for generating aggregated signatures across multiple Solana keyholders, enabling distributed signing without exposing private keys. It uses the MuSig2 protocol (via the ZenGo multi-party-eddsa library) to combine partial signatures from n-of-n signers into a single valid Solana transaction.

The API supports three transaction types:
- Native SOL transfers
- SPL token transfers
- Stake account operations (create, deactivate, withdraw)

## Features

- **Two-round TSS signing protocol**: Step One generates partial nonces, Step Two produces partial signatures
- **Key aggregation**: Combine multiple pubkeys into a single aggregated public key
- **Native SOL transfers**: Single-signer and multi-party aggregated transactions
- **SPL token transfers**: ERC-20 style token transfers with multi-party signing
- **Stake account management**: Create, delegate, deactivate, and withdraw stake with TSS
- **Base58 serialization**: All TSS messages and signatures transmitted as base58-encoded strings
- **Multi-network support**: Mainnet, Testnet, and Devnet endpoints

**Modules**:
- `main.rs` - HTTP handlers and route definitions (Poem)
- `tss.rs` - Core MuSig2 implementation: key_agg, step_one, step_two, signature aggregation
- `serialization.rs` - Base58 encoding/decoding for TSS data structures
- `staking.rs` - Stake account transaction builders
- `spl_token_utils.rs` - SPL token transaction builders
- `models.rs` - Request/response structs for all API endpoints
- `error.rs` - Error types and conversions

## Installation

```toml
[dependencies]
solana-tss-api-backend = "0.1.0"
```

Or from source:
```bash
git clone <repository>
cargo build --release
```

## Quick Example

Generate an aggregated key from multiple participants:

```bash
# Step 1: Each participant generates their keypair share
curl -X GET http://localhost:8000/api/generate
# Returns: {"secret_share": "...", "public_share": "..."}

# Step 2: Aggregate public keys
curl -X POST http://localhost:8000/api/aggregate_keys \
  -H "Content-Type: application/json" \
  -d '{"keys": ["<pubkey1>", "<pubkey2>", "<pubkey3>"]}'
# Returns: {"aggregated_public_key": "..."}

# Step 3: Each participant generates message 1 (nonce)
curl -X POST http://localhost:8000/api/agg_send_step_one \
  -H "Content-Type: application/json" \
  -d '{"keypair": "<base58_keypair>"}'
# Returns: {"message_1": "...", "secret_state": "..."}

# Step 4: Each participant generates partial signature
curl -X POST http://localhost:8000/api/agg_send_step_two \
  -H "Content-Type: application/json" \
  -d '{
    "keypair": "<base58_keypair>",
    "amount": 1.0,
    "to": "<recipient_pubkey>",
    "recent_block_hash": "<hash>",
    "keys": ["<pubkey1>", "<pubkey2>", "<pubkey3>"],
    "first_messages": ["<msg1>", "<msg2>", "<msg3>"],
    "secret_state": "<secret>"
  }'
# Returns: {"partial_signature": "..."}

# Step 5: Aggregate all partial signatures and broadcast
curl -X POST http://localhost:8000/api/aggregate_signatures \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 1.0,
    "to": "<recipient_pubkey>",
    "recent_block_hash": "<hash>",
    "net": "devnet",
    "keys": ["<pubkey1>", "<pubkey2>", "<pubkey3>"],
    "signatures": ["<sig1>", "<sig2>", "<sig3>"]
  }'
# Returns: {"transaction_id": "..."}
```

## Usage Notes

- **Async runtime**: Requires Tokio (configured with `features = ["full"]`)
- **Network selection**: All requests targeting RPC require `net` field: `mainnet`, `testnet`, or `devnet`
- **Base58 encoding**: Keys, signatures, and TSS messages use base58 for JSON compatibility
- **TSS round coordination**: External orchestration required to collect partial signatures from all participants
- **Blockhash expiry**: Recent blockhash must be valid; client must fetch fresh blockhash before each transaction
- **SPL tokens**: Requires specifying `token_mint` and `decimals` for token transfers
- **Stake accounts**: Uses seed-derived addresses with `create_with_seed`

## Security

** UNAUDITED **. This code implements cryptographic threshold signatures but has not undergone external security review. Production use requires:

- Independent cryptographic audit
- Secure key distribution for participant shares
- Network-level transport security (HTTPS/TLS)
- Rate limiting and request validation
- Input sanitization beyond basic parsing

Use on mainnet at your own risk.

