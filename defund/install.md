# DeFund

## Requirments
* 16GB RAM
* 4vCPUs
* 200GB Disk space
* Ubuntu 20.04 or higher
* Go v1.19.1

## Step 1 - Preparation server

### How to update apt

```
sudo apt update && apt upgrade -y
```

### How to install pakage
```
sudo apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-
```

### How to install go

```
curl -s https://raw.githubusercontent.com/DOUBLE-TOP/tools/main/go.sh | bash
```

## Step 2 - Installation node

###  Clone working repo and install node
```
git clone https://github.com/defund-labs/defund
cd defund && git checkout v0.1.0
make install
```
### Check version
`defundd version --long | grep -e version -e commit`

 ```
 0.1.0
 commit: 05390d047b3de40fbe23572e02ade4cd282a75ed
 ```

## Step 3 - Preparation Node

###  Download genesis
```
wget -O $HOME/.defund/config/gensis.tar.gz https://raw.githubusercontent.com/defund-labs/testnet/main/defund-private-3/defund-private-3-gensis.tar.gz

tar -xzvf gensis.tar.gz

rm -rf gensis.tar.gz
```

#### Check genesis
`sha256sum $HOME/.defund/config/genesis.json`

 ```
 1a10121467576ab6f633a14f82d98f0c39ab7949102a77ab6478b2b2110109e3
 ```


###  Download Adrrbook

```
wget -O $HOME/.defund/config/addrbook.json "http://65.108.6.45:8000/defund/addrbook.json"
```
###  Config node

```
defundd config chain-id defund-private-3
```

### Set the minimum price for gas in app.toml
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025ufetf\"/;" ~/.defund/config/app.toml
```

### Add external_address/seeds/peers to config.toml

```
external_address=$(wget -qO- eth0.me)
seeds="85279852bd306c385402185e0125dffeed36bf22@38.146.3.194:26656"
peers="6366ac3af3995ecbc48c13ce9564aef0c7a6d7df@defund-testnet.nodejumper.io:28656,03d46eae18d935a2e820735563ab01abb17d4cb6@65.108.235.107:29656,081a38c22f5c1915c3c38b529ef112370b45e290@161.97.91.80:26656"
```

```
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.defund/config/config.toml
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/"$HOME.defund/config/config.toml
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.defund/config/config.toml
```
### (Optional) Setup custom pruning

```
pruning="custom"
pruning_keep_recent="100"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.defund/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.defund/config/app.toml
```

### (Optional) Setup custom port for node

#### In this case we use ports

```
10658 10657 10656 10660 10657
1060 1090 1091 1017
```

```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:10658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:10657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:1061\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:10656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":10660\"%" $HOME/.defund/config/config.toml

sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:1090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:1191\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:1117\"%" $HOME/.defund/config/app.toml

sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:10657\"%" $HOME/.defund/config/client.toml
```

### (Optional) State Sync

```
peers="5e7853ec4f74dba1d3ae721ff9f50926107efc38@65.108.6.45:60556"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.defund/config/config.toml
```

```
SNAP_RPC=https://t-defund.rpc.utsa.tech:443

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.defund/config/config.toml
```

## Step 4 - Run Node

#### Create a service file

```
sudo tee /etc/systemd/system/defundd.service > /dev/null <<EOF
[Unit]
Description=defundd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which defundd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

```
systemctl daemon-reload
systemctl enable defundd
systemctl restart defundd && journalctl -u defundd -f -o cat
```

## Well done! Check useful comand:
