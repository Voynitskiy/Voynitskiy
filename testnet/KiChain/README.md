# Ki Chain [kichain-t-4]
![KiChain Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/KiChain/KiChain.png)
### Links
* `Website` - https://foundation.ki/
* `Medium` - https://medium.com/ki-foundation
* `Twitter` - https://twitter.com/Ki_Foundation
* `Telegram` - https://t.me/kifoundation
* `Discord` - https://discord.gg/aj99cT65ND
### Links testnet
* https://github.com/KiFoundation/ki-networks/tree/v0.1/Testnet/kichain-t-4
* https://github.com/KiFoundation/ki-networks/blob/v0.1/Testnet/kichain-t-4/UPGRADE_V3.md
* https://github.com/KiFoundation/ki-networks/blob/v0.1/Testnet/kichain-t-4/UPGRADE_t3_t4.md
### RPC
* `RPC` 65.109.28.177:21157
### Peers and seeds
* `Peer` cd106a09cbb727791e649c0ab7c1985a0db5eb8b@65.109.28.177:21156
* `Peers` https://github.com/KiFoundation/ki-networks/blob/v0.1/Testnet/kichain-t-4/peers.txt
* `Seeds` https://github.com/KiFoundation/ki-networks/blob/v0.1/Testnet/kichain-t-4/seeds.txt
### Archive folders wasm
* `wasm` `01 sep 2022` https://github.com/Voynitskiy/Voynitskiy/raw/main/testnet/KiChain/wasm.tar.gz
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/KiFoundation/ki-networks/v0.1/Testnet/kichain-t-4/genesis.json
* `Addrbook` https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/testnet/KiChain/addrbook.json
### Explorer
* `Explorer` https://kichain-t-4.blockchain.ki
## Installation Steps
>Prerequisite: go1.16+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current Testnet ki-tools version.
```shell
git clone https://github.com/KiFoundation/ki-tools.git
cd ki-tools
git checkout -b v3.0.0-beta tags/3.0.0-beta
make build-testnet
cp build/kid $HOME/go/bin/
```
* Init
```
kid init <moniker> --chain-id kichain-t-4
kid config chain-id kichain-t-4
```

### Generate keys
```
kid keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/KiFoundation/ki-networks/v0.1/Testnet/kichain-t-4/genesis.json > ~/.kid/config/genesis.json
curl https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/testnet/KiChain/addrbook.json > ~/.$COSMOS/config/addrbook.json
```
### Peers, seed
```
seeds="381dff5439ed042353c5333e61bab1510711f2f5@seed-testnet.blockchain.ki:6969"
peers="46b25d81510f8dcc535ca0924961b266e4f59244@135.125.183.94:26656,ada3bbf64f963e764bfe003276354bd121e80ae0@95.111.248.200:26656,276f6fb420b3595b63c2a13d35868cb530a31578@65.21.159.19:26656,7e5710ee0b1576a78a21a89e1588b6c95ee69873@194.163.137.193:26656,323a5c9ccfb73573cbcd634c497b2a7405b198fa@142.132.137.114:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.kid/config/config.toml
```
* Minimum gas prices
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025utki\"/;" ~/.kid/config/app.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/kid.service > /dev/null <<EOF
[Unit]
Description=Ki Chain Testnet
After=network-online.target

[Service]
User=$USER
ExecStart=$(which kid) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable kid
sudo systemctl restart kid && journalctl -fu kid -o cat
```
## Create Validator
```
kid tx staking create-validator \
  --amount=1000000utki \
  --pubkey=$(kid tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="kichain-t-4" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas-prices=0.025utki \
  --from=<wallet_name>
```
### State-Sync
* download wasm
```
cd && cd .kid && \
wget https://github.com/Voynitskiy/Voynitskiy/raw/main/testnet/KiChain/wasm.tar.gz && \
rm -R wasm \
tar -xvf wasm.tar.gz && \
rm -R wasm.tar.gz
```
* start with State-Sync
```
SNAP_RPC=65.109.28.177:21157 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop kid && kid tendermint unsafe-reset-all --home $HOME/.kid
```
```
peers="cd106a09cbb727791e649c0ab7c1985a0db5eb8b@65.109.28.177:21156"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.kid/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.kid/config/config.toml
```
```
sudo systemctl restart kid && journalctl -fu kid -o cat
```
