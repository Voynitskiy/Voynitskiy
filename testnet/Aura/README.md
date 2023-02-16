# Aura [euphoria-2]
![Aura Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/testnet/Aura/Aura.png)
### Validator AlxVoy
* `Delegate` https://test.anode.team/aura/staking/auravaloper10dpsdmx2jm32fe4sudg79l9jrh5aep8l344gjn
* `Valoper` auravaloper10dpsdmx2jm32fe4sudg79l9jrh5aep8l344gjn
### Links
* `Website` - https://aura.network/
* `Twitter` - https://twitter.com/AuraNetworkHQ
* `Discord` - https://discord.gg/am96YqMyjk
* `Github` https://github.com/aura-nw/aura
* `Telegram Channel` https://t.me/auranw
* `Telegram Group Global` https://t.me/AuraNetworkOfficial
* `Linkedin` https://www.linkedin.com/company/auranetwork/
* `Facebook` https://facebook.com/AuraNetworkHQ/
* `Whitepaper` https://github.com/aura-nw/whitepaper/blob/main/release/Aura_Network___whitepaper.pdf
### RPC & API
* `RPC` https://aura.rpc.t.anode.team
* `API` https://aura.api.t.anode.team
### Peers and seeds
* `Peer` 6ef01ca6714aa8127d1b21b5339909ca6319dae0@144.76.97.251:26776
* `Live peers` https://anode.team/Aura/test/peers.txt
### Genesis and addrbook
* `Genesis` https://anode.team/Aura/test/genesis.json
* `Addrbook` https://anode.team/Aura/test/addrbook.json
### Explorer
* `ANODE.TEAM` https://test.anode.team/aura
### Auto installation script
```
wget -O Aura.sh https://anode.team/Aura/test/Aura.sh && chmod u+x Aura.sh && ./Aura.sh
```
## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/aura-nw/aura
cd aura
git checkout euphoria_v0.4.2
make install
```
* Init
```
aurad init <moniker> --chain-id euphoria-2
aurad config chain-id euphoria-2
```

### Generate keys
```
aurad keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://anode.team/Aura/test/genesis.json > ~/.aura/config/genesis.json
curl https://anode.team/Aura/test/addrbook.json > ~/.aura/config/addrbook.json
```
### Peers, seed
```
peers="6ef01ca6714aa8127d1b21b5339909ca6319dae0@144.76.97.251:26776"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.aura/config/config.toml
```
### Add min gas
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ueaura\"/" $HOME/.aura/config/app.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/aurad.service > /dev/null <<EOF
[Unit]
Description=Aura
After=network-online.target

[Service]
User=$USER
ExecStart=$(which aurad) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable aurad
sudo systemctl restart aurad && journalctl -fu aurad -o cat
```
## Create Validator
```
aurad tx staking create-validator \
  --amount=1000000ueaura \
  --pubkey=$(aurad tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="euphoria-2" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://aura.rpc.t.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop aurad && aurad tendermint unsafe-reset-all --home $HOME/.aura --keep-addr-book
```
```
peers="6ef01ca6714aa8127d1b21b5339909ca6319dae0@144.76.97.251:26776"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.aura/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.aura/config/config.toml
```
```
curl -o - -L https://anode.team/Aura/test/anode.team_aura_wasm.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.aura
```
```
sudo systemctl restart aurad && journalctl -fu aurad -o cat
```
