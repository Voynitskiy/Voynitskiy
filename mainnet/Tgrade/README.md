# Tgrade [tgrade-mainnet-1]
![Tgrade Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/Tgrade/Tgrade.png)
### Validator AlxVoy
* `Delegate` https://tgrade.aneka.io/validators/tgrade1q598gmmltugefxxxp76z3re8sedw5gdxfe0wck
* `Valoper` tgrade1q598gmmltugefxxxp76z3re8sedw5gdxfe0wck
### Links
* `Website` - https://tgrade.finance/
* `Medium` - https://medium.com/tgradefinance
* `Twitter` - https://twitter.com/tgradefinance
* `GitHyb` - https://github.com/confio/tgrade-networks
* `YouTube` - https://www.youtube.com/channel/UC0trSt_EboEYZNO7xSRE2eA
* `Discord` - https://discord.gg/pHF2QY73Hh
* `Dapp` - https://dapp.tgrade.finance
### Links mainnet
* https://github.com/confio/tgrade-networks
### RPC
* `RPC` https://tgrade.rpc.m.anode.team
### API
* `API` https://tgrade.api.m.anode.team
### Peers and seeds
* `Peer` deab21ea29a24c50699ca3691b50dfd54b2a3a93@144.76.97.251:28826
* `Peers` https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/Tgrade/peers.txt
* `Seeds` https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/Tgrade/seeds.txt
### Archive folders wasm
* `wasm` `04 sep 2022` https://github.com/Voynitskiy/Voynitskiy/raw/main/mainnet/Tgrade/wasm.tar.gz
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/confio/tgrade-networks/main/mainnet-1/config/genesis.json
* `Addrbook` https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/Tgrade/addrbook.json
### Explorer
* `Mintscan` https://www.mintscan.io/tgrade
* `Aneka` 	https://tgrade.aneka.io
## Installation Steps
>Prerequisite: go1.18.3+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/confio/tgrade
cd tgrade
git checkout v2.0.2
make build
sudo mv build/tgrade /usr/local/bin
```
* Init
```
tgrade init <moniker> --chain-id tgrade-mainnet-1
tgrade config chain-id tgrade-mainnet-1
```

### Generate keys
```
tgrade keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/confio/tgrade-networks/main/mainnet-1/config/genesis.json > ~/.tgrade/config/genesis.json
curl https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/Tgrade/addrbook.json > ~/.tgrade/config/addrbook.json
```
### Peers, seed
```
seeds="0c3b7d5a4253216de01b8642261d4e1e76aee9d8@45.76.202.195:26656,8639bc931d5721a64afc1ea52ca63ae40161bd26@194.163.144.63:26656"
peers="0a63421f67d02e7fb823ea6d6ceb8acf758df24d@142.132.226.137:26656,4a319eead699418e974e8eed47c2de6332c3f825@167.235.255.9:26656,6918efd409684d64694cac485dbcc27dfeea4f38@49.12.240.203:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.tgrade/config/config.toml
```
* Minimum gas prices
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.05utgd\"/;" ~/.tgrade/config/app.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/tgrade.service > /dev/null <<EOF
[Unit]
Description=Tgrade
After=network-online.target

[Service]
User=$USER
ExecStart=$(which tgrade) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable tgrade
sudo systemctl restart tgrade && journalctl -fu tgrade -o cat
```
## Create Validator
```
tgrade tx poe create-validator \
  --amount=0utgd \
  --vesting-amount=1000000utgd \
  --pubkey=$(tgrade tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="tgrade-mainnet-1" \
  --node https://rpc.mainnet-1.tgrade.confio.run:443 \
  --fees 200000utgd \
  --gas auto \
  --gas-adjustment 1.4 \
  --from=<wallet_name>
```
### State-Sync
* download wasm
```
cd && cd .tgrade && \
wget https://github.com/Voynitskiy/Voynitskiy/raw/main/mainnet/Tgrade/wasm.tar.gz && \
rm -R wasm \
tar -xvf wasm.tar.gz && \
rm -R wasm.tar.gz
```
* start with State-Sync
```
SNAP_RPC=https://tgrade.rpc.m.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop kid && kid tendermint unsafe-reset-all --home $HOME/.tgrade
```
```
peers="deab21ea29a24c50699ca3691b50dfd54b2a3a93@144.76.97.251:28826"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.tgrade/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.tgrade/config/config.toml
```
```
sudo systemctl restart tgrade && journalctl -u tgrade -f
```
