# GenesisL1 [genesis_29-2]
![GenesisL1 Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/GenesisL1/GenesisL1.png)
### Validator AlxVoy
* `Delegate` https://ping.pub/genesisl1/staking/genesisvaloper1jxyxqzs0eg53gf2vjs5ekx7ewgduf4gqcpay30
* `Valoper` genesisvaloper1jxyxqzs0eg53gf2vjs5ekx7ewgduf4gqcpay30
### Links
* `Website` - https://genesisl1.com/
* `Twitter` - https://twitter.com/genesis_L1
* `Telegram` - https://t.me/GenesisL1
* `Discord` - https://discord.gg/aZAG4bawVn
* `GitHub` - https://github.com/alpha-omega-labs/genesisd
### Links mainnet
* https://github.com/alpha-omega-labs/genesisd
### RPC
* `RPC` 144.76.97.251:21497
### Peers and seeds
* `Peer` eefdbf7eb40265a9e900a10d36de1c088b49420e@144.76.97.251:21496
* `Peers` https://github.com/alpha-omega-labs/genesisd/blob/neolithic/peers_list.txt
### Genesis and addrbook
* `Genesis` https://github.com/alpha-omega-labs/genesisd/raw/neolithic/genesis_29-1-state/genesis.json
* `Addrbook` https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/GenesisL1/addrbook.json
### Explorer
* `Ping` https://ping.pub/genesisL1
* `Atomscan` https://atomscan.com/genesisl1
## Installation Steps
>Prerequisite: go1.17+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current Testnet ki-tools version.
```shell
git clone https://github.com/alpha-omega-labs/genesisd
cd genesisd
make install
```
* Init
```
genesisd init <moniker> --chain-id genesis_29-2
genesisd config chain-id genesis_29-2
```

### Generate keys
```
genesisd keys add <wallet_name>
```
### Genesis, addrbook
```
wget -O $HOME/.genesisd/config/genesis.json "https://github.com/alpha-omega-labs/genesisd/raw/neolithic/genesis_29-1-state/genesis.json"
curl https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/GenesisL1/addrbook.json > ~/.genesisd/config/addrbook.json
```
### Peers, seed
```
seeds="36111b4156ace8f1cfa5584c3ccf479de4d94936@65.21.34.226:26656"
peers="5082248889f93095a2fd4edd00f56df1074547ba@146.59.81.204:26651,36111b4156ace8f1cfa5584c3ccf479de4d94936@65.21.34.226:26656,c23b3d58ccae0cf34fc12075c933659ff8cca200@95.217.207.154:26656,37d8aa8a31d66d663586ba7b803afd68c01126c4@65.21.134.70:26656,d7d4ea7a661c40305cab84ac227cdb3814df4e43@139.162.195.228:26656,be81a20b7134552e270774ec861c4998fabc2969@genesisl1.3ventures.io:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.genesisd/config/config.toml
```
* Minimum gas prices
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0el1\"/;" ~/.genesisd/config/app.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/genesisd.service > /dev/null <<EOF
[Unit]
Description=GenesisL1
After=network-online.target

[Service]
User=$USER
ExecStart=$(which genesisd) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable genesisd
sudo systemctl restart genesisd && journalctl -fu genesisd -o cat
```
## Create Validator
```
genesisd tx staking create-validator \
  --amount=1000000el1 \
  --pubkey=$(genesisd tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="genesis_29-2" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=144.76.97.251:21497 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop genesisd && genesisd unsafe-reset-all --home $HOME/.genesisd
```
```
peers="eefdbf7eb40265a9e900a10d36de1c088b49420e@144.76.97.251:21496"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.genesisd/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.genesisd/config/config.toml
```
```
sudo systemctl restart genesisd && journalctl -u genesisd -f
```
