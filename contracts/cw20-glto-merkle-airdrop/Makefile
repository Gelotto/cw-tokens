network ?= devnet  # network := devnet|mainnet|testnet
contract_addr_filepath ?= $(release_dirpath)/contract_addr.txt
release_dirpath ?= ./release

# build optimized WASM artifact
build:
	./bin/build

# deploy to local dev validator, assuming it's running
deploy:
	./bin/deploy ./artifacts/cw20_base.wasm $(network)

# instantiate last contract to be deployed
instantiate:
	# `minter` is the address of the admin, owner, and minter
	./bin/instantiate $(network) $(minter)

# run all unit tests
test:
	RUST_BACKTRACE=1 cargo unit-test

# Generate the contract's JSONSchema JSON files in schemas/
schemas:
	cargo schema

# Run/start local "devnet" validator docker image	
validator:
	./bin/validator

mint:
	# mint some token for the given recipient address
	./bin/client mint $(network) $(contract_addr_filepath) $(recipient) $(amount)

get-balance:
	# get token balance for the given wallet address
	./bin/client balance $(network) $(contract_addr_filepath) $(address)

get-token-info:
	# get token balance for the given wallet address
	./bin/client info $(network) $(contract_addr_filepath)

get-minter:
	# get minter info (address, supply cap)
	./bin/client minter $(network) $(contract_addr_filepath)