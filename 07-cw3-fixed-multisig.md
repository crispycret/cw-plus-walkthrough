# CW3 Fixed Multsig




# STORE:

```
BASE_OPTIONS="--chain-id testing --gas-prices 0.1ujunox --gas auto --gas-adjustment 1.3 -b block -y"

# Manually look for Code ID value
junod tx wasm store cw3_fixed_multisig.wasm --from master $BASE_OPTIONS --output json | jq -r ''

# OR

CW3_FIXED_MULTISIG_CODE_ID=$(\
  junod tx wasm store cw3_fixed_multisig.wasm --from master $BASE_OPTIONS \
  --output json | jq -r '.logs[0].events[-1].attributes[0].value')
  
echo $CW3_FIXED_MULTISIG_CODE_ID
```



# INSTANTIATE:
### Building a query using the instantiate schema - `cw3-fixed-multisg/schema/instantiate_msg.json`
```
'{
    "max_voting_period": {
        "time": 500000
    },
    
    "required_weight": 2,
    
    "voters": [
        {
            "addr": "<addr>",
            "weight": 1
        }, 
        {
            "addr": "<addr>",
            "weight": 1
        }, 
        {
            "addr": "<addr>",
            "weight": 1
        }
    ]
}'
```

### Format the instantiate message into JSON
There are 3 voters each with a weight of 1 and the vote requires a weight of 2 two pass. Two of the Three voters must vote Yes.
```
'{"max_voting_period":{"time":500000},"required_weight":2,"voters":[{"addr":"<address>","weight":1},{"addr":"<address>","weight":1},{"addr":"<address>","weight":1}]}'
```

### Incorporate your addresses into the message
```
echo $ADMIN_A
echo $ADMIN_B
echo $ADMIN_C

# output
ADMIN_A = juno1rxjp2u9kxf7aqssxzcguuhvemwxlj0rj5tkgyh
ADMIN_B = juno1d3pdywztwek0v5sxqa8t6kwq7fkqfrl8y94tqw
ADMIN_C = juno1mt5we8478ud82auklx5mcj8mjxjxsh808kqgtm

# JSON of my 3 admins having the same voting power.
'{"max_voting_period":{"time":500000},"required_weight":2,"voters":[{"addr":"juno1rxjp2u9kxf7aqssxzcguuhvemwxlj0rj5tkgyh","weight":1},{"addr":"juno1d3pdywztwek0v5sxqa8t6kwq7fkqfrl8y94tqw","weight":1},{"addr":"juno1mt5we8478ud82auklx5mcj8mjxjxsh808kqgtm","weight":1}]}'
```

# Store JSON message as a variable and instantiate the contract
```
INSTANTIATE_MSG='{"max_voting_period":{"time":500000},"required_weight":2,"voters":[{"addr":"juno1rxjp2u9kxf7aqssxzcguuhvemwxlj0rj5tkgyh","weight":1},{"addr":"juno1d3pdywztwek0v5sxqa8t6kwq7fkqfrl8y94tqw","weight":1},{"addr":"juno1mt5we8478ud82auklx5mcj8mjxjxsh808kqgtm","weight":1}]}'

CW3_FIXED_MULTISIG_CONTRACT_ADDRESS=$(junod tx wasm instantiate $CW3_FIXED_MULTISIG_CODE_ID $INSTANTIATE_MSG --label "cw3 fixed multisig" --from admin-a $BASE_OPTIONS --output json | jq -r '.logs[0].events[0].attributes[0].value')
```


## (Optional) Store the variable as a persisent environment variable
For the purposes of testing I like to store the cotract address that I will be using as an environment variable that persists through multiple session. Add the following line with your contract address to the bottom `~/.profile`

#### Display then copy your contract address
```
echo $CW3_FIXED_MULTISIG_CONTRACT_ADDRESS
```

#### Store the outputed value in `~/.profile`
```
CW3_FIXED_MULTISIG_CONTRACT_ADDRESS=<contract address>
```



# QUERY:

### Query Instantiations of Contracts
```
junod q wasm list-contracts-by-code-id $CW3_FIXED_MULTISIG_CODE_ID
```


### Query Messages for cw3-fixed-multisig
```
threshold - "Return ThresholdResponse"
'{"threshold":{}}'


proposal - Returns ProposalResponse
'{"proposal":{proposal_id:1}}'


list_proposals - Returns ProposalListResponse
'{"list_proposals":{"limit":1,"start_after":1}}


reverse_proposals - Returns ProposalListResponse
'{"reverse_proposals":{"limit":1,"start_before":1}}'


vote - Returns VoteResponse
'{"vote":{"proposal_id":1,"voter":"juno1rxjp2u9kxf7aqssxzcguuhvemwxlj0rj5tkgyh"}}'


list_votes - Returns VoteListResponse
'{"list_votes":{"proposal_id":1}}'


list_votes - Returns VoteListResponse (With Optional Parameters)
'{"list_votes":{"proposal_id":1,"limit":1,"start_after":null}}'


voter - Returns VoterInfo
'{"voter":{"address":"juno1rxjp2u9kxf7aqssxzcguuhvemwxlj0rj5tkgyh"}}'


list_voters - Returns VoterListResponse
'{"list_voters":{}}'


list_voters - Returns VoterListResponse (with optional parameters)
'{"list_voters":{"limit":1,"start_after":null}}'
```


## Query Commands
```
junod query wasm contract-state smart $CW3_FIXED_MULTISIG_CONTRACT_ADDRESS '{"threshold":{}}' --chain-id testing

junod query wasm contract-state smart $CW3_FIXED_MULTISIG_CONTRACT_ADDRESS '{"list_voters":{}}' --chain-id testing

junod query wasm contract-state smart $CW3_FIXED_MULTISIG_CONTRACT_ADDRESS '{"list-proposals":{}}' --chain-id testing

junod query wasm contract-state smart $CW3_FIXED_MULTISIG_CONTRACT_ADDRESS '{"proposal":{"proposal_id":1}}' --chain-id testing

junod query wasm contract-state smart $CW3_FIXED_MULTISIG_CONTRACT_ADDRESS '{"voter":{proposal_id:1}}' --chain-id testing


```





# EXECUTE:

The CW3 Fixed Multisig contract gives us enough functionallity to finally start to experimenting with major blockchain fundementals. In this case limited goverance.
While instantiating the contract we decieded that their will be 3 admins, each with a weight of 1 and that a weight of 2 is required to pass a proposal. Let's dimenstrate that now with a proposal that does nothing.

### Execute Messages for cw3-fixed-multisig
```

# submit proposal

'{"propose": {"description": "Test Proposal Description", "msgs": "[]", "title": "Test Proposal"}}'

### schema showing optional arguments
{
  "propose": {
    "description": "",
    "msgs": [CosmosMsg_for_Empty],
    "title": "",
    "latest": : Expiration (One Of) -> [
        {"at_height": BlockHeight -> 200},
        {"at_time":[] Timestamp -> UINT64: A point in time in nanosecond precision. },
        {"never": {}}},
    ]
  }
}


##### latest Schema
"latest": {
    "anyOf": [
        {
            "$ref": "#/definitions/Expiration"
        },
        {
            "type": "null"
        }
    ]
},
```


# Create a proposal, view it and vote on it.

## Submit an empty proposal
We know the proposal we are about to submit does nothing because it has no messages.

```
junod tx wasm execute $CW3_FIXED_MULTISIG_CONTRACT_ADDRESS \
  '{"propose":{"description":"Testing","msgs":[],"title":"Testing Proposal"}}' \
  --from admin-a $BASE_OPTIONS
```

### Lookup the proposal we have just submitted
```
junod query wasm contract-state smart $CW3_FIXED_MULTISIG_CONTRACT_ADDRESS '{"proposal":{"proposal_id":1}}' --chain-id testing
```





## [Next Chapter - 08 CW3 Flex Multisig](08-cw3-flex-multisig.md)


## [Previous Chapter - 06 CW1 Subkeys.md](06-cw1-subkeys.md)
