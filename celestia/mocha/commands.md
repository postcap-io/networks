# Useful Commands celestia-appd

## Check Node commands:

### Check logs:
`journalctl -n 100 -f -u celestia-appd`

### Check status systmd:  
`systemctl status celestia-appd`

### Check status node:
`celestia-appd status 2>&1 | jq`

### Check balances:
* addr - your wallet address
`celestia-appd q bank balances <addr>`

### Check validator:
```
celestia-appd query staking validator <valoper_address>
```

```
celestia-appd query staking validators --limit 1000000 -o json | jq '.validators[] | select(.description.moniker=="<name_moniker>")' | jq
```

### Check how many blocks are skipped by the validator
```
celestia-appd q slashing signing-info $(celestia-appdd tendermint show-validator)
```

### Check activ set validators

```
celestia-appd q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

### Check inactiv set validators

```
celestia-appd q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

## Wallet commands :

### List wallets

```
celestia-appd keys list
```

### Show account key

```
celestia-appd keys show <name_wallet> --bech acc
```

### Show validator key

```
celestia-appd keys show <name_wallet> --bech val
```

### Show consensus key

```
celestia-appd keys show <name_wallet> --bech cons
```

### Show all supported addresses

```
celestia-appd debug addr <wallet_addr>
```

### Show private key

```
celestia-appd keys export <name_wallet> --unarmored-hex --unsafe
```

### Account request

```
celestia-appd q auth account $(celestia-appdd keys show <name_wallet> -a) -o text
```

### Delete wallet

```
celestia-appdd keys delete <name_wallet>
```


## Transaction:

### Collect rewards from all validators, items delegated (no commission)

```
celestia-appdd tx distribution withdraw-all-rewards --from <name_wallet> --fees 5000ufetf -y
```

### Collect rewards + commission from your own validator

```
celestia-appdd tx distribution withdraw-rewards <valoper_address> --from <name_wallet> --fees 5000ufetf --commission -y
```

### Delegate more to your stake (this is how 1 coin is sent)

```
celestia-appdd tx staking delegate <valoper_address> 1000000ufetf --from <name_wallet> --fees 5000ufetf -y
```

### Redelegation to another validator

```
celestia-appdd tx staking redelegate <src-validator-addr> <dst-validator-addr> 1000000ufetf --from <name_wallet> --fees 5000ufetf -y
```

### Unbond

```
celestia-appdd tx staking unbond <addr_valoper> 1000000ufetf --from <name_wallet> --fees 5000ufetf -y
```

### Send coins to another address

```
celestia-appdd tx bank send <name_wallet> <address> 1000000ufetf --fees 5000ufetf -y
```

### Unjail

```
celestia-appdd tx slashing unjail --from <name_wallet> --fees 5000ufetf -y
```



## Governance :

#### List proposals

```
celestia-appdd q gov proposals
```

### See the result of the vote

```
celestia-appdd q gov proposals --voter <ADDRESS>
```

### Vote for the proposal

```
celestia-appdd tx gov vote 1 yes --from <name_wallet> --fees 555ufetf
```

### Make a deposit to the offer

```
celestia-appdd tx gov deposit 1 1000000ufetf --from <name_wallet> --fees 555ufetf
```

### Create an offer

```
celestia-appdd tx gov submit-proposal --title="Randomly reward" --description="Reward 10 testnet participants who completed more than 3 tasks" --type="Text" --deposit="11000000grain" --from=<name_wallet> --fees 500grain
```

## Peers and RPC :

### Find out your peer

```
PORTR=$(grep -A 3 "\[p2p\]" ~/.celestia-appd/config/config.toml | egrep -o ":[0-9]+") && \
echo $(celestia-appdd tendermint show-node-id)@$(curl ifconfig.me)$PORTR
```

### Find out RPC port

```
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.celestia-appd/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

### Checking the number of peers

```
curl -s localhost:10657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr | split(":")[2])"' | wc -l
```

### List of monikers of connected peers

```
curl -s localhost:10657/net_info | jq '.result.peers[].node_info.moniker'
```

### Checking prevotes/precommits
```
curl -s localhost:10657/consensus_state | jq '.result.round_state.height_vote_set[0].prevotes_bit_array' && \
curl -s localhost:10657/consensus_state | jq '.result.round_state.height_vote_set[0].precommits_bit_array'
```

### Check prevote of your validator

```
curl -s localhost:10657/consensus_state -s | grep $(curl -s localhost:10657/status | jq -r .result.validator_info.address[:12])
```

## Delete node

```
systemctl stop celestia-appdd && \
systemctl disable celestia-appdd && \
rm /etc/systemd/system/celestia-appdd.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .celestia-appd celestia-appd && \
rm -rf $(which celestia-appdd)
```
