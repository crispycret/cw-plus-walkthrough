cw-plus work notes

### Resource commands
`junod tx bank send [from_key_or_address] [to_address] [amount] [flags]`
`junod query bank balances [address] [flags]`

`junod tx bank send $(junod keys show unsafe-test -a) juno1uzaa2sexws4gatetng5ke0lrqpfy89khd990u9 40000000ujuno --generate-only`

### Denomiations and values
The Metric System prefix u is micro or millionths which is 0.000001 of the base


UNSURE ABOUT THIS PART
We start out with 1000 juno.

1 juno = 1000000 ujunox
10 juno = 10000000 ujunox
100 juno = 1000000000 ujunox
1000 juno = 10000000000 ujunox

### Keys
#### Top
unsafe-test: `juno16g2rahf5846rxzp3fwlswy08fz8ccuwk03k57y`

master: `juno1uzaa2sexws4gatetng5ke0lrqpfy89khd990u9`
```water remember cushion abuse banner rice sheriff image army siege pony wink strong cart knee vocal used cheese choose during foam mango erode lamp```

reserve: `juno1vrau67dz6xej65kzjvqw24jpxy27uvhs8t06hj`
```opera supply sunny add actor mercy bus camera stage mind spare win code clap critic know athlete vibrant endless access circle sunny ozone taxi```



#### Admins
admin-a: `juno10pfa9a5l8sy0czqjy7tlquyhrmjn90yhr50562`
```fall three item coconut lizard climb liar escape midnight subway trial million father upper subject develop view hand sauce radar coast deliver water joke```

admin-b: `juno1hry3jzq5vca4uufl79cvqqmjqe53qfaxkxuc3u`
```bus umbrella rose follow gallery brain inflict solar ceiling picnic monitor announce merge remain foot brass panic accuse dilemma taste goddess share warm inspire```

admin-c: `juno1ayw38tapu8wd3l57fwdhwekcymhcueh59p2pa8`
```civil nasty s#hallow tank base pony exact fatigue thought subject water nest hobby way result bind sport film daughter lawsuit silk rally shiver market```


#### Users
user-a: `juno1f7qvu4gzjc3c7vzxl845shtzwewqdd8hdm7ql7`
```sun uphold butter city coin ski guitar mention yellow broom beyond assume satoshi dragon monster whale announce bulk hawk drink scale resemble render desert```

user-b: `juno14dnyyawm4c3pr88fscnqu5ppt5dp0cq7tum5xy`
```actual science notice elder unit reject erosion clinic artefact rose renew mystery swamp frame moon raven saddle eye helmet fatal mansion must six crumble```

user-c: `juno1pz65jurpjjfmhwusqhv8d9mruyyty4845wm767`
```dune shy loop water knife leaf baby toe clean lobster general wine joke fantasy leaf used century lens midnight enjoy ball similar athlete radio```

user-d: `juno1ujmkzdn0cz83ysvssu30nxsx7jfqrc0hkp94sp`
```whip globe lock keep fever east short trumpet once bulb melody quality spoil empower curve plastic express where system luggage cake urban juice tube```


#### As variables
export admin-a



# Getting Started

Before starting this series we must do the following.

### Install junod

set the moniker name and chain id.

moniker name can be anything
chain id must be `testing` to work on localhost

run the juno docker image that preloads the unsafe-test key with juno balance.

```
docker run -it \
  -p 26656:26656 \
  -p 26657:26657 \
  -e STAKE_TOKEN=ujunox \
  -e UNSAFE_CORS=true \
  ghcr.io/cosmoscontracts/juno:v2.1.0 \
  ./setup_and_run.sh juno16g2rahf5846rxzp3fwlswy08fz8ccuwk03k57y
```

Add the unsafe-test key if needed
`junod keys add unsafe-test --recover`

Seed: `clip hire initial neck maid actor venue client foam budget lock catalog sweet steak waste crater broccoli pipe steak sister coyote moment obvious choose`

using the unsafe-test key send 10 juno to our master key.

    Create another key with name master that we will us to store and instantiate contracts with.
`junod keys add master`

    send 10 juno to the master key
    
`junod tx bank send unsafe-test juno1uzaa2sexws4gatetng5ke0lrqpfy89khd990u9 10000000ujunox --chain-id testing`

    this master key will be the master user for most contracts.
    we will experiment with setting a master admin upon instantiation. In those cases the master key will not be the master of that contract, just used to store the contract.

Create a reserve key that also hold 10 juno. This account will be used as a faucet for the admin and user accounts.
'juno keys add reserve`

`junod tx bank send unsafe-test juno1vrau67dz6xej65kzjvqw24jpxy27uvhs8t06hj 10000000ujunox --chain-id testing`

The seperation between the two above keys will give us an idea of how much we have spent from storing, instantiating and executing contracts


`junod query bank balances juno16g2rahf5846rxzp3fwlswy08fz8ccuwk03k57y`
`junod query bank balances juno1uzaa2sexws4gatetng5ke0lrqpfy89khd990u9`
`junod query bank balances juno1vrau67dz6xej65kzjvqw24jpxy27uvhs8t06hj`


create 3 more keys each with an implied lower lever of permissions for some some circumstances and to be equal admins for multisig like circumstances.
    admin-a, admin-b, admin-c
    Allocate keys with 5, 2.5, 1.5 juno respectivley from the reserve key
    We should send 9 juno to key-a, from key-a send 4 juno to key-b, from key-b send 1.5 juno to key-c

`junod tx bank send reserve juno10pfa9a5l8sy0czqjy7tlquyhrmjn90yhr50562 9000000ujunox --chain-id testing`

`junod query bank balances juno1vrau67dz6xej65kzjvqw24jpxy27uvhs8t06hj`

`junod query bank balances juno10pfa9a5l8sy0czqjy7tlquyhrmjn90yhr50562`

`junod tx bank send admin-a juno1hry3jzq5vca4uufl79cvqqmjqe53qfaxkxuc3u 4000000ujunox --chain-id testing`
`junod query bank balances juno1hry3jzq5vca4uufl79cvqqmjqe53qfaxkxuc3u`

`junod tx bank send admin-b juno1ayw38tapu8wd3l57fwdhwekcymhcueh59p2pa8 1500000ujunox --chain-id testing`
`junod query bank balances juno1ayw38tapu8wd3l57fwdhwekcymhcueh59p2pa8`


create 4 more keys to be used as user level accounts.
    user-a, user-b, user-c, user-d
    from the reserve key send each user 0.25 juno

`junod tx bank send reserve juno1f7qvu4gzjc3c7vzxl845shtzwewqdd8hdm7ql7 250000ujunox --chain-id testing`
`junod tx bank send reserve juno14dnyyawm4c3pr88fscnqu5ppt5dp0cq7tum5xy 250000ujunox --chain-id testing`
`junod tx bank send reserve juno1pz65jurpjjfmhwusqhv8d9mruyyty4845wm767 250000ujunox --chain-id testing`
`junod tx bank send reserve juno1ujmkzdn0cz83ysvssu30nxsx7jfqrc0hkp94sp 250000ujunox --chain-id testing`

`junod query bank balances juno1vrau67dz6xej65kzjvqw24jpxy27uvhs8t06hj`
`junod query bank balances juno1f7qvu4gzjc3c7vzxl845shtzwewqdd8hdm7ql7`
`junod query bank balances juno14dnyyawm4c3pr88fscnqu5ppt5dp0cq7tum5xy`
`junod query bank balances juno1pz65jurpjjfmhwusqhv8d9mruyyty4845wm767`
`junod query bank balances juno1ujmkzdn0cz83ysvssu30nxsx7jfqrc0hkp94sp`

# Refill the reserve with 10 juno
`junod tx bank send unsafe-test juno1vrau67dz6xej65kzjvqw24jpxy27uvhs8t06hj 10000000ujunox --chain-id testing`


Now for learning.

first downland the cw-plus repo

build all contracts
