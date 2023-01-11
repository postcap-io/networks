# Install Celestia Mocha

## Requirments
* 8GB RAM
* Quad-Core
* 250GB SSD Disk space
* Ubuntu 20.04 or higher
* Go v1.19.1
* Bandwidth: 1 Gbps for Download/100 Mbps for Upload

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
git clone https://github.com/celestiaorg/celestia-app.git
cd celestia-app && git checkout v0.11.0
make install
```
### Check version
`celestia-appd version --long | grep -e version -e commit`

 ```
 0.11.0
 commit: 281a352a03e23be27e7c0b9e2e40677d0cf90be5
 ```
### Initialize the node
#### Replace <MONIKER> to your Moniker Name
`celestia-appd init <MONIKER> --chain-id mocha`

## Step 3 - Preparation Node

###  Download genesis
```
wget -O $HOME/.celestia-app/config/gensis.json https://raw.githubusercontent.com/postcap-io/networks/main/celestia/mocha/genesis.json
```

#### Check genesis
`sha256sum $HOME/.celestia-app/config/genesis.json`

 ```
 05ef265e16f37d1f5aa2ec884be3782c38d71e59a6d57957235c5ca433aa8e05
 ```


###  Download Adrrbook

```
wget -O $HOME/.celestia-app/config/addrbook.json "https://raw.githubusercontent.com/postcap-io/networks/main/celestia/mocha/addrbook.json"
```
###  Config node

```
celestia-appd config chain-id mocha
```

### Set the minimum price for gas in app.toml
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.005utia\"|" $HOME/.celestia-app/config/app.toml
```

### Add external_address/seeds/peers to config.toml

```
external_address=$(wget -qO- eth0.me)
seeds="3f472746f46493309650e5a033076689996c8881@celestia-testnet.rpc.kjnodes.com:20659"
```

```
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.celestia-app/config/config.toml
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/"$HOME/.celestia-app/config/config.toml
```
### (Optional) Setup custom pruning

```
pruning="custom"
pruning_keep_recent="100"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.celestia-app/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.celestia-app/config/app.toml
```

### (Optional) Setup custom port for node

#### In this case we use ports

```
10658 10657 10656 10660 10657
1060 1090 1091 1017
```

```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:10658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:10657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:1061\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:10656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":10660\"%" $HOME/.celestia-app/config/config.toml

sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:1090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:1191\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:1117\"%" $HOME/.celestia-app/config/app.toml

sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:10657\"%" $HOME/.celestia-app/config/client.toml

celestia-appd config node tcp://localhost:10657
```

### (Optional) State Sync

```
peers="d5519e378247dfb61dfe90652d1fe3e2b3005a5b@celestia-testnet.rpc.kjnodes.com:20656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.celestia-app/config/config.toml
```

```
SNAP_RPC=https://celestia-testnet.rpc.kjnodes.com:443

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.celestia-app/config/config.toml
```

## Step 4 - Run Node

#### Create a service file

```
sudo tee /etc/systemd/system/celestia-appd.service > /dev/null <<EOF
[Unit]
Description=celestia-appd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which celestia-appd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

```
systemctl daemon-reload
systemctl enable celestia-appd
systemctl restart celestia-appd && journalctl -u celestia-appd -f -o cat
```

## Well done! Check validator.md and useful commands.md!
