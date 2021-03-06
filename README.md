# CW Plus Walkthrough

Walkthrough each contract in the cw-plus repository. See `01-GettingStarted` for instructions on getting a testnet up and running.  

This repository is sloppy as I am still learning and organizing the new information that I learn. I am experimenting with organizing the information in different patterns as I do not yet know how to present this data. Expect major changes frequently if using this resource.  

Built from the [offical juno documentation](https://docs.junonetwork.io/smart-contracts-and-junod-development/tutorial-cw1/download-compile-store) and other crediting sources at the bottom.

# Table Of Contents

### Setup
#### [Chapter 01 Getting Started](01-GettingStarted.md)
#### [Chapter 02 Accounts](01-GettingStarted.md)
#### [Chapter 03 Funding](03-Funding.md)

### Cw-Plus Contracts
#### [Chapter 04 Building Contracts](04-BuildingContracts.md)
#### [Chapter 05 CW1 Whitelist](05-cw1-whitelist.md)
#### [Chapter 06 CW1 Subkeys](06-cw1-subkeys.md)
#### [Chapter 07 CW3 Fixed Multisig](07-cw3-fixed-multisig.md)
#### [Chapter 08 CW3 Flex Multisig.md](08-cw3-flex-multisig.md)


# 

## Follow the Table of Contents, reading further is NOT Required and is meant to be a quick start / summary of the table of contents.
 


### Things to Know:

#### Denomiations and values
The Metric System prefix u is micro or millionths which is 0.000001 of the base

##### Unsure of this

We start with 1000000000 ujunox or 100 juno.

```
1 juno = 1000000 ujunox
10 juno = 10000000 ujunox
100 juno = 1000000000 ujunox
```


## Resource commands
```
# Create, show remove, accounts
junod keys add <keyname>
junod keys show <keyname> -a
junod keys delete <keyname>

# Check balance, send, check tx
junod q bank balances <address>
junod tx bank send <key_or_address> <to_address> <amount_and_denom> [flags]
junod q tx <TX_HASH>

# Show number of contracts stored on-chain
junod q wasm list-code

# Show the number of instantiations of A contract as a list of contract addresses
junod q wasm list-contract-by-code <code-id>
junod query wasm list-contract-by-code 1
```




## Quick Start
For PRACTICE ONLY use `password` as the password for all accounts. This will speed things along while you learn.

#### Replace Genisis File
```
curl https://raw.githubusercontent.com/CosmosContracts/testnets/main/uni-2/genesis.json > ~/.juno/config/genesis.json
```

#### Import Public Unsafe Test Key
```
juno keys add unsafe-test --recover
```
#### Unsafe Test Key's bip39 mnemonic (Copy)
```
clip hire initial neck maid actor venue client foam budget lock catalog sweet steak waste crater broccoli pipe steak sister coyote moment obvious choose
```

#### Setup and Basic Commands
###### Creating a Master Account
```
junod keys add master
MASTER=$(junod keys show master -a)
junod query bank balances $MASTER
```

###### Sending juno from the unsafe-test account to the new master account.
```
junod tx bank send unsafe-test $MASTER 10000000ujunox
junod q bank balances $MASTER 

UNSAFE_TEST=$(junod keys show unsafe-test -a)
junod q bank balances $UNSAFE_TEST 

SEND_TX=$(junod tx bank send unsafe-test $MASTER 10000000ujunox | jq -r '.txhash')

junod q tx $SEND_TX
```


## Build All Contracts
Navigate to the root folder of the cw-plus folder and run the following command to build all contracts at once.

```
sudo docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/code/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/workspace-optimizer:0.11.3
```




## Store A Contract
##### Using Smart Contract ([CW1 Subkeys](06-cw1-subkeys.md) as Example)
This can only be done after building the contracts. Navigate to the `artifacts` folder inside the cw-plus repo.

```
cd artifacts

CW1_SUBKEYS_STORE_TX=$(junod tx wasm store cw1_subkeys.wasm  --from master --chain-id=testing --gas auto --output json -y | jq -r '.txhash')
CW1_SUBKEYS_CODE_ID=$(junod query tx $CW1WHITELIST_STORE_TX --output json | jq -r '.code')
```

## Instantiate A Contract
```
CW1_SUBKEYS_CONTRACT_ADDRESS=$(junod tx wasm instantiate $CW1_SUBKEYS_CODE_ID "{\"admins\":[\"$MASTER\", \"$ADMIN_A\"],\"mutable\":false}" --amount 50000ujunox --label "CW1 example contract" --from test --chain-id testing \
  --gas-prices 0.1ujunox --gas auto --gas-adjustment 1.3 -b block -y --output json | jq -r ".logs[0].events[2].attributes[0].value"))
```

## Querying A Contract
```
junod query wasm contract-state smart $CW1_WHITELIST_CONTRACT_ADDRESS '{"admin_list":{}}' --chain-id testing
```


## Send an Execute Message to A Contract
```
junod tx wasm execute $CW1_WHITELIST_CONTRACT_ADDRESS '{"update_admins": {"admins":["<ADDR>", "<ADDR>", "<ADDR>"]}}' --from <key> --chain-id testing --gas-prices 0.1ujunox --gas auto --gas-adjustment 1.3 -b block
```




# Walkthough Summary:


# CW1 Whitelist:

### Descripion:

This contract allows admins of a contract to be set. If the contract variable `mutable=true` then admins can be added or removed. If `mutable=false` then the admins that were set at the instantiation of the contract will be the only admins allowed for this contract and `mutable` cannot be set to `true`

### Store
```

BASE_OPTIONS="--chain-id testing --gas-prices 0.1ujunox --gas auto --gas-adjustment 1.3 -b block -y"

CW1_WHITELIST_STORE_TX=$(junod tx wasm store cw1_whitelist.wasm --from master $BASE_OPTIONS --output json | jq -r '.txhash')

CW1_WHITELIST_CODE_ID=$(junod query tx $CW1_WHITELIST_STORE_TX --output json | jq -r '.logs[0].events[-1].attributes[0].value')

echo $CW1_WHITELIST_CODE_ID
```

### Instantiate
Baking `INSTANTIATE_MSG` into the command does not work yet.
```
INSTANTIATE_MSG=$("{\"admins\":[\"$ADMIN_A\", \"$ADMIN_B\"], \"mutable\":true}")

CW1_WHITELIST_CONTRACT_ADDRESS=$(junod tx wasm instantiate $CW1_WHITELIST_CODE_ID "{\"admins\":[\"$ADMIN_A\", \"$ADMIN_B\"], \"mutable\":true}" $INSTANTIATE_OPTIONS --from master $BASE_OPTIONS --output json | jq -r '.logs[0].events[2].attributes[0].value')

echo $CW1_WHITELIST_CONTRACT_ADDRESS
```

### Query
```
QUERY_MSG='{"admin_list":{}}'
junod query wasm contract-state smart $CW1_WHITELIST_CONTRACT_ADDRESS $QUERY_MSG --chain-id testing
```

### Execute
```
EXECUTE_MSG = "{\"update_admins\": {\"admins\":[\"$ADMIN_A\", \"$ADMIN_B\", \"$ADMIN_C\"]}}" 

junod tx wasm execute $CW1_WHITELIST_CONTRACT_ADDRESS $EXECUTE_MSG --from admin-a $BASE_OPTIONS 
```



# CW1 Subkeys:

### Descripion:

This contract incorporartes cw1-whitelist and therefore has all of the same functionality. It expands functionality by allowing allowances of a native token to be set by admins.

### Store
```
```

### Instantiate
```
junod tx wasm instantiate $CW1SUBKEYS_CODE '{"admins":["$admin_a", "$admin_b"], "mutable":true}' --amount 50000ujunox --label "cw1-whitelist" --from master --chain-id testing --gas-prices 0.1ujunox --gas auto --gas-adjustment 1.3 -b block -y

cd artifacts
TX=$(junod tx wasm store cw1_subkeys.wasm  --from <your-key> --chain-id=<chain-id> --gas auto --output json -y | jq -r '.txhash')
CODE_ID=$(junod query tx $TX --output json | jq -r '.logs[0].events[-1].attributes[0].value')
```

### Query
```
```

### Execute
```
```



# CW3 Fixed Multisig

### Descripion:
Description of CW2 Fixed Multisig

### Store
```
```

### Instantiate
```
```

### Query
```
```

### Execute
```
```






# CW3 Flex Multisig

### Descripion:
Description of CW2 Fixed Multisig

### Store
```
```

### Instantiate
```
```

### Query
```
```

### Execute
```
```



