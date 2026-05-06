# TSS API

Solana threshold signature server using MuSig2 multi-party signing.

## Run

```bash
cargo build --release
./target/release/tss
```

Server runs on `http://127.0.0.1:8000`.

## API

| Endpoint | Description |
|---|---|
| `POST /api/generate` | Generate keypair |
| `POST /api/balance` | Get SOL balance |
| `POST /api/airdrop` | Request airdrop (testnet/devnet) |
| `POST /api/send_single` | Single-signer SOL transfer |
| `POST /api/aggregate_keys` | Combine pubkeys into agg key |
| `POST /api/agg_send_step_one` | Generate nonce (step 1) |
| `POST /api/agg_send_step_two` | Generate partial sig (step 2) |
| `POST /api/aggregate_signatures` | Combine sigs & broadcast |
| `POST /api/spl_token_balance` | Get SPL token balance |
| `POST /api/spl_send_single` | Single-signer token transfer |
| `POST /api/spl_agg_send_step_two` | Token partial sig |
| `POST /api/spl_aggregate_signatures` | Combine token sigs & broadcast |
| `POST /api/stake` | Create stake account |
| `POST /api/deactivate_stake` | Deactivate stake |
| `POST /api/withdraw_stake` | Withdraw stake |
| `POST /api/agg_stake_step_two` | Stake partial sig |
| `POST /api/aggregate_stake_signatures` | Combine stake sigs & broadcast |

## Flow

```
1. aggregate_keys([pubkey1, pubkey2, ...]) -> agg_pubkey
2. Each signer: agg_send_step_one(keypair) -> (message_1, secret_state)
3. Each signer: agg_send_step_two(keypair, amount, to, blockhash, keys, messages, secret) -> partial_signature
4. aggregate_signatures(amount, to, blockhash, keys, signatures) -> transaction_id
```

## Networks

`mainnet`, `testnet`, `devnet` via `net` field in requests.

## Key Formats

- Keypairs: base58
- Pubkeys: base58
- TSS messages: base58
- Blockhash: base58