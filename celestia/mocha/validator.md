# Run Validator

### Set up wallet

`lavad keys add <name_wallet> `

##### Restore wallet (after insert seed command)
`lavad keys add <name_wallet> --recover`

##### After creating a wallet, you need to go to the project's discord, request tokens and check the balance of the wallet. Tokens are needed to create a validator
`celestia-appd q bank balances <addr>`

```
celestia-appd tx staking create-validator \
--chain-id mocha \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1000000" \
--amount 1000000ufetf \
--pubkey $(celestia-appd tendermint show-validator) \
--moniker "<name_moniker>" \
--from <name_wallet>
```
