CW3 - Fixed Multsig




STORE:

junod tx wasm store cw3_fixed_multisig.wasm --from master --chain-id testing \
  --gas-prices 0.1ujunox --gas auto --gas-adjustment 1.3 -b block -y
  
junod tx wasm store cw1_subkeys.wasm  --from test --chain-id testing \
  --gas-prices 0.1ujunox --gas auto --gas-adjustment 1.3 -b block -y




INSTANTIATE:

QUERY:

EXECUTE:





