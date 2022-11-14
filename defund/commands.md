# Useful Commands DeFund

## Check Node commands:

### Check logs:
`journalctl -n 100 -f -u defundd`

### Check status systmd:  
`systemctl status defundd`

### Check status node:
`defundd status 2>&1 | jq`

### Check balances:
* addr - your wallet address
`defundd q bank balances <addr>`

### Check validator:
```
defundd query staking validator <valoper_address>
```

```
defundd query staking validators --limit 1000000 -o json | jq '.validators[] | select(.description.moniker=="<name_moniker>")' | jq
```

### Check how many blocks are skipped by the validator
```
defundd q slashing signing-info $(defundd tendermint show-validator)
```

### Check activ set validators

```
defundd q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

### Check inactiv set validators

```
defundd q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

## Wallet commands :

### List wallets

```
defundd keys list
```

### Show account key

```
defundd keys show <name_wallet> --bech acc
```

### Show validator key

```
defundd keys show <name_wallet> --bech val
```

### Show consensus key

```
defundd keys show <name_wallet> --bech cons
```

### Show all supported addresses

```
defundd debug addr <wallet_addr>
```

### Show private key

```
defundd keys export <name_wallet> --unarmored-hex --unsafe
```

### Account request

```
defundd q auth account $(defundd keys show <name_wallet> -a) -o text
```

### Delete wallet

```
defundd keys delete <name_wallet>
```


## Transaction:

### Collect rewards from all validators, items delegated (no commission)

```
defundd tx distribution withdraw-all-rewards --from <name_wallet> --fees 5000ufetf -y
```

### Collect rewards + commission from your own validator

```
defundd tx distribution withdraw-rewards <valoper_address> --from <name_wallet> --fees 5000ufetf --commission -y
```

### Delegate more to your stake (this is how 1 coin is sent)

```
defundd tx staking delegate <valoper_address> 1000000ufetf --from <name_wallet> --fees 5000ufetf -y
```

### Redelegation to another validator

```
defundd tx staking redelegate <src-validator-addr> <dst-validator-addr> 1000000ufetf --from <name_wallet> --fees 5000ufetf -y
```

### Unbond

```
defundd tx staking unbond <addr_valoper> 1000000ufetf --from <name_wallet> --fees 5000ufetf -y
```

### Send coins to another address

```
defundd tx bank send <name_wallet> <address> 1000000ufetf --fees 5000ufetf -y
```

### Unjail

```
defundd tx slashing unjail --from <name_wallet> --fees 5000ufetf -y
```



## Governance :

#### List proposals

```
defundd q gov proposals
```

### See the result of the vote

```
defundd q gov proposals --voter <ADDRESS>
```

### Vote for the proposal

```
defundd tx gov vote 1 yes --from <name_wallet> --fees 555ufetf
```

### Make a deposit to the offer

```
defundd tx gov deposit 1 1000000ufetf --from <name_wallet> --fees 555ufetf
```

### Create an offer

```
defundd tx gov submit-proposal --title="Randomly reward" --description="Reward 10 testnet participants who completed more than 3 tasks" --type="Text" --deposit="11000000grain" --from=<name_wallet> --fees 500grain
```

## Peers and RPC :

### Find out your peer

```
PORTR=$(grep -A 3 "\[p2p\]" ~/.defund/config/config.toml | egrep -o ":[0-9]+") && \
echo $(defundd tendermint show-node-id)@$(curl ifconfig.me)$PORTR
```

### Find out RPC port

```echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/.defund/config/config.toml | egrep -o ":[0-9]+")\033[0m"
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
curl -s localhost:10657/consensus_state -s | grep $(curl -s localhost:26657/status | jq -r .result.validator_info.address[:12])
```

## Delete node

```
systemctl stop defundd && \
systemctl disable defundd && \
rm /etc/systemd/system/defundd.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .defund defund && \
rm -rf $(which defundd)
```
