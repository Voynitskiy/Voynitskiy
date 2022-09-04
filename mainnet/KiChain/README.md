# Ki Chain [kichain-2]
### Validator AlxVoy
* `Delegate` https://ping.pub/kichain/staking/kivaloper1tm2p6va7gpsy0ettqk583pl5a05d39vml3rtk5
* `Valoper` kivaloper1tm2p6va7gpsy0ettqk583pl5a05d39vml3rtk5
### Links
* `Website` - https://foundation.ki/
* `Medium` - https://medium.com/ki-foundation
* `Twitter` - https://twitter.com/Ki_Foundation
* `Telegram` - https://t.me/kifoundation
* `Discord` - https://discord.gg/aj99cT65ND
### Links mainnet
* https://github.com/KiFoundation/ki-networks/tree/v0.1/Mainnet/kichain-2
* https://github.com/KiFoundation/ki-networks/blob/v0.1/Mainnet/kichain-2/UPGRADE_V3.md
* https://github.com/KiFoundation/ki-networks/blob/v0.1/Mainnet/kichain-2/UPGRADE_kichain-1_to_2.md
### RPC
* `RPC` 65.108.12.222:26637
### Peers and seeds
* `Peer` 6dbcc6a1726bb7030875f3a60718dddc0c6f5de2@65.108.12.222:26636
* `Peers` https://github.com/KiFoundation/ki-networks/blob/v0.1/Mainnet/kichain-2/peers.txt
* `Seeds` https://github.com/KiFoundation/ki-networks/blob/v0.1/Mainnet/kichain-2/seeds.txt
### Archive folders wasm
* `wasm` `01 sep 2022` https://github.com/Voynitskiy/Voynitskiy/raw/main/mainnet/KiChain/wasm.tar.gz
### Genesis and addrbook
* `Genesis` https://github.com/KiFoundation/ki-networks/raw/v0.1/Mainnet/kichain-2/genesis.json
* `Addrbook` https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/KiChain/addrbook.json
### Explorer
* `Mintscan` https://www.mintscan.io/ki-chain
* `Ping` https://ping.pub/kichain
* `Atomscan` https://atomscan.com/ki-chain
* `The code dev` https://ki.thecodes.dev/
## Installation Steps
>Prerequisite: go1.16+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current Testnet ki-tools version.
```shell
wget https://github.com/KiFoundation/ki-tools/releases/download/3.0.0/kid-mainnet-3.0.0-linux-amd64
mv kid-mainnet-3.0.0-linux-amd64 kid
chmod u+x kid
mv kid /root/go/bin/
```
* Init
```
kid init <moniker> --chain-id kichain-2
kid config chain-id kichain-2
```

### Generate keys
```
kid keys add <wallet_name>
```
### Genesis, addrbook
```
wget -O $HOME/.kid/config/genesis.json "https://github.com/KiFoundation/ki-networks/raw/v0.1/Mainnet/kichain-2/genesis.json"
curl https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/KiChain/addrbook.json > ~/.kid/config/addrbook.json
```
### Peers, seed
```
seeds="2088a9e86b4cb5140bf0c2830d4ab6824c85c4aa@seed.blockchain.ki:6969"
peers="450af457247b59aa558a26a14bd7ac4bf86eeae70@195.201.164.223:26656,81eef39d2ca9a07490857d197423da4ba5e01879@15.188.134.35:26656,5adb5ad6a6fcef624866cefdb551dafdc07f7e78@15.188.198.188:26656,41b321292cbe50c5c30017cc71c404481be0e20b@3.38.12.5:26656,644df8ae7f92e4b77cce887479798b7a7b300797@162.55.189.153:26656,f2b80411c2b48935b796c91c907565c3bd78aff4@142.132.184.154:26656,90c0614a1af1320665cab280bd5e73a18ddf09b8@38.242.200.186:26656,520f6bef2b8fa29dc618b080fe99767562089c78@65.108.206.131:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.kid/config/config.toml
```
* Minimum gas prices
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025uxki\"/;" ~/.kid/config/app.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/kid.service > /dev/null <<EOF
[Unit]
Description=Ki Chain
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
  --amount=1000000uxki \
  --pubkey=$(kid tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="kichain-2" \
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
wget https://github.com/Voynitskiy/Voynitskiy/raw/main/mainnet/KiChain/wasm.tar.gz && \
rm -R wasm \
tar -xvf wasm.tar.gz && \
rm -R wasm.tar.gz
```
* start with State-Sync
```
SNAP_RPC=65.108.12.222:26637 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop kid && kid tendermint unsafe-reset-all --home $HOME/.kid
```
```
peers="6dbcc6a1726bb7030875f3a60718dddc0c6f5de2@65.108.12.222:26636"
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
