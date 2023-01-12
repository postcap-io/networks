# Run Validator

### Set up wallet

`celestia-appd keys add <name_wallet> `

##### Restore wallet (after insert seed command)
`celestia-appd keys add <name_wallet> --recover`

##### After creating a wallet, you need to go to the project's discord, request tokens and check the balance of the wallet. Tokens are needed to create a validator
`celestia-appd q bank balances <addr>`

```
celestia-appd tx staking create-validator \
--chain-id mocha \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1000000" \
--amount 1000000utia \
--pubkey $(celestia-appd tendermint show-validator) \
--moniker "<name_moniker>" \
--evm-address="evm_addr" \
--orchestrator-address="your_orchestrator_addr" \
--gas-adjustment=1.4 \
--gas=auto \
--fees=5000utia \
--from <name_wallet>
```
> `--moniker`: Name your Moniker
> `--from`: Name your wallet
> `--evm-address`: This flag should contain a 0x EVM address. Here, you can add any Ethereum-based address to this flag. You can also modify it later if you decide to switch addresses.
>
> `--orchestrator-address`: This flag should contain a newly-generated celestia1 Celestia address. Validators certainly can use their existing Celestia addresses here but it is recommended to create a new one.
