# CW20 Merkle Airdrop

This is a merkle airdrop smart contract that works with cw20 token specification Mass airdrop distributions made cheap
and efficient.

Explanation of merkle
airdrop: [Medium Merkle Airdrop: the Basics](https://medium.com/smartz-blog/merkle-airdrop-the-basics-9a0857fcc930)

Traditional and non-efficient airdrops:

- Distributor creates a list of airdrop
- Sends bank send messages to send tokens to recipients

**Or**

- Stores list of recipients on smart contract data
- Recipient claims the airdrop

These two solutions are very ineffective when recipient list is big. First, costly because bank send cost for the
distributor will be costly. Second, whole airdrop list stored in the state, again costly.

Merkle Airdrop is very efficient even when recipient number is massive.

This contract works with multiple airdrop rounds, meaning you can execute several airdrops using same instance.

Uses **SHA256** for merkle root tree construction.

## Procedure

- Distributor of contract prepares a list of addresses with many entries and publishes this list in public static .js
  file in JSON format
- Distributor reads this list, builds the merkle tree structure and writes down the Merkle root of it.
- Distributor creates contract and places calculated Merkle root into it.
- Distributor says to users, that they can claim their tokens, if they owe any of addresses, presented in list,
  published on distributor's site.
- User wants to claim his N tokens, he also builds Merkle tree from public list and prepares Merkle proof, consisting
  from log2N hashes, describing the way to reach Merkle root
- User sends transaction with Merkle proof to contract
- Contract checks Merkle proof, and, if proof is correct, then sender's address is in list of allowed addresses, and
  contract does some action for this use.
- Distributor sends token to the contract, and registers new merkle root for the next distribution round.

## Spec

### Messages

#### InstantiateMsg

`InstantiateMsg` instantiates contract with owner and cw20 token address. Airdrop `stage` is set to 0.

```rust
pub struct InstantiateMsg {
  pub owner: String,
  pub cw20_token_address: String,
}
```

#### ExecuteMsg

```rust
pub enum ExecuteMsg {
  UpdateConfig {
    owner: Option<String>,
  },
  RegisterMerkleRoot {
    merkle_root: String,
  },
  Claim {
    stage: u8,
    amount: Uint128,
    proof: Vec<String>,
  },
}
```

- `UpdateConfig{owner}` updates configuration.
- `RegisterMerkleRoot {merkle_root}` registers merkle tree root for further claim verification. Airdrop `Stage`
  increased by 1.
- `Claim{stage, amount, proof}` recipient executes for claiming airdrop with `stage`, `amount` and `proof` data built
  using full list.

#### QueryMsg

``` rust
pub enum QueryMsg {
    Config {},
    MerkleRoot { stage: u8 },
    LatestStage {},
    IsClaimed { stage: u8, address: String },
}
```

- `{ config: {} }` returns configuration, `{"cw20_token_address": ..., "owner": ...}`.
- `{ merkle_root: { stage: "1" }` returns merkle root of given stage, `{"merkle_root": ... , "stage": ...}`
- `{ latest_stage: {}}` returns current airdrop stage, `{"latest_stage": ...}`
- `{ is_claimed: {stage: "stage", address: "wasm1..."}` returns if address claimed airdrop, `{"is_claimed": "true"}`

## Merkle Airdrop CLI

[Merkle Airdrop CLI](helpers) contains js helpers for generating root, generating and verifying proofs for given airdrop
file.

## Test Vector Generation

Test vector can be generated using commands at [Merkle Airdrop CLI README](helpers/README.md)
# CW20 Basic (GLTO Token)

## Release Steps

### Environment Variables:

- `$NETWORK`: Name of network, namely `devnet|testnet|mainnet`.
- `$CW20_TOKEN_CONTRACT`: address of GLTO CW20 token contract instance.

### Steps

#### 1. Deploy Contract

- Run `make network=$NETWORK build deploy`

#### 2. Instantiate Contract

- Run `make network=$NETWORK cw20_token_address=$CW20_TOKEN_ADDRESS instantiate`

#### 3. Generate & Set Merkle Root

- Ensure `receivers.json` is up-to-date to generate `./release/merkle-root.txt`
- Run `make generate-merkle-root`
- Run `make network=$NETWORK set-merkle-root-in-contract`.

This is a basic implementation of a cw20 contract. It implements
the [CW20 spec](../../packages/cw20/README.md) and is designed to
be deployed as is, or imported into other contracts to easily build
cw20-compatible tokens with custom logic.

Implements:

- [x] CW20 Base
- [x] Mintable extension
- [x] Allowances extension

## Running this contract

You will need Rust 1.44.1+ with `wasm32-unknown-unknown` target installed.

You can run unit tests on this via:

`cargo test`

Once you are happy with the content, you can compile it to wasm via:

```
RUSTFLAGS='-C link-arg=-s' cargo wasm
cp ../../target/wasm32-unknown-unknown/release/cw20_base.wasm .
ls -l cw20_base.wasm
sha256sum cw20_base.wasm
```

Or for a production-ready (optimized) build, run a build command in the
the repository root: https://github.com/CosmWasm/cw-plus#compiling.

## Importing this contract

You can also import much of the logic of this contract to build another
ERC20-contract, such as a bonding curve, overiding or extending what you
need.

Basically, you just need to write your handle function and import
`cw20_base::contract::handle_transfer`, etc and dispatch to them.
This allows you to use custom `ExecuteMsg` and `QueryMsg` with your additional
calls, but then use the underlying implementation for the standard cw20
messages you want to support. The same with `QueryMsg`. You _could_ reuse `instantiate`
as it, but it is likely you will want to change it. And it is rather simple.

Look at [`cw20-staking`](https://github.com/CosmWasm/cw-tokens/tree/main/contracts/cw20-staking) for an example of how to "inherit"
all this token functionality and combine it with custom logic.
