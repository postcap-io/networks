# Run Validator

```
defundd tx staking create-validator \
--chain-id defund-private-3 \
--commission-rate 0.05 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1000000" \
--amount 1000000ufetf \
--pubkey $(defundd tendermint show-validator) \
--moniker "<name_moniker>" \
--from <name_wallet>
```
