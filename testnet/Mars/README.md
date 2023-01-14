# Mars [ares-1]
![Mars Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/testnet/Mars/Mars.png)
### Validator AlxVoy
* `Delegate` https://test.anode.team/mars/staking/marsvaloper1yc53leq759uh23yzkgmsv4mr2n359773jvduws
* `Valoper` marsvaloper1yc53leq759uh23yzkgmsv4mr2n359773jvduws
### Links
* `Website` - https://marsprotocol.io/
* `Twitter` - https://twitter.com/mars_protocol
* `Discord` - https://discord.gg/marsprotocol
* `Github` https://github.com/mars-protocol
* `Telegram` https://t.me/martiannews
* `Medium` https://mars-protocol.medium.com/
* `Forum` https://forum.marsprotocol.io/
* `Blog` https://blog.marsprotocol.io/
* `Youtube` https://www.youtube.com/channel/UCKcwNg4deLUrHAX74zS0ozw
* `Reddit` https://www.reddit.com/r/marsprotocol_io/
* `Docs` https://docs.marsprotocol.io/mars-protocol
* `Web App` https://app.marsprotocol.io/
* `Whitepaper` https://github.com/mars-protocol/whitepaper/blob/main/README.md
### RPC
* `RPC` https://mars.rpc.t.anode.team
### API
* `API` https://mars.api.t.anode.team
### Peers and seeds
* `Peer` b7937ceadfd3b89c5232c744832c3f7f918642a1@65.109.28.177:30657
### Genesis and addrbook
* `Genesis` https://anode.team/Mars/test/genesis.json
* `Addrbook` https://anode.team/Mars/test/addrbook.json
### Explorer
* `ANODE.TEAM` https://test.anode.team/mars
### Auto installation script
```
wget https://anode.team/Mars/test/setup_mars.sh && chmod u+x setup_mars.sh && ./setup_mars.sh
```
## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/mars-protocol/hub mars
cd mars
git checkout v1.0.0-rc7
make install
```
* Init
```
marsd init <moniker> --chain-id ares-1
marsd config chain-id ares-1
```

### Generate keys
```
marsd keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://anode.team/Mars/test/genesis.json > ~/.mars/config/genesis.json
curl https://anode.team/Mars/test/addrbook.json > ~/.mars/config/addrbook.json
```
### Peers, seed
```
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:18556"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.mars/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/marsd.service > /dev/null <<EOF
[Unit]
Description=Mars
After=network-online.target

[Service]
User=$USER
ExecStart=$(which marsd) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable marsd
sudo systemctl restart marsd && journalctl -fu marsd -o cat
```
## Create Validator
```
marsd tx staking create-validator \
  --amount=5000000umars \
  --pubkey=$(marsd tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="ares-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://mars.rpc.t.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop marsd && marsd tendermint unsafe-reset-all --home $HOME/.mars --keep-addr-book
```
```
peers="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:18556"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.mars/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.mars/config/config.toml
```
```
sudo systemctl restart marsd && journalctl -fu marsd -o cat
```
