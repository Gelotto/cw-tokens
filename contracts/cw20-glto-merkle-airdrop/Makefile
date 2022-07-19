network ?= devnet  # network := devnet|mainnet|testnet
release_dirpath ?= ./release
contract_addr_filepath ?= $(release_dirpath)/contract_addr.txt
merkle_root_filepath ?= $(release_dirpath)/merkle-root.txt
sender ?= juno16g2rahf5846rxzp3fwlswy08fz8ccuwk03k57y

# build optimized WASM artifact
build:
	./bin/build

# deploy to local dev validator, assuming it's running
deploy:
	./bin/deploy $(network) $(sender) ./artifacts/cw20_glto_merkle_airdrop.wasm

# instantiate last contract to be deployed
instantiate:
	./bin/instantiate $(network) $(sender) $(cw20_token_address)

# run all unit tests
test:
	RUST_BACKTRACE=1 cargo unit-test

# Generate the contract's JSONSchema JSON files in schemas/
schemas:
	cargo schema

# Run/start local "devnet" validator docker image	
validator:
	./bin/validator

# generate merkel tree for the given asset (osmo, juno, etc) and 
# write root hash to file in ./merkle-tree directory.
generate-merkle-root:
	mkdir -p $(release_dirpath) && ./bin/merkle generate-merkle-root ../airdrop.json '$(merkle_root_filepath)'

# upload merkle root hash to airdrop contract
set-merkle-root-in-contract:
	./bin/merkle set-merkle-root-in-contract $(network) $(sender) '$(contract_addr_filepath)' '$(merkle_root_filepath)'

# check if the given address has already claimed
has-claimed:
	./bin/merkle has-claimed $(network) '$(contract_addr_filepath)' '$(address)'
