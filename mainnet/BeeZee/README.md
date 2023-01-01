# BeeZee [beezee-1]
![BeeZee Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/BeeZee/BeeZee.png)
### Validator AlxVoy
* `Delegate` https://main.anode.team/beezee/staking/bzevaloper17excul8pv3pj974z4u60unthpg0t2777n37n08
* `Valoper` bzevaloper17excul8pv3pj974z4u60unthpg0t2777n37n08
### Links
* `Website` - https://getbze.com/
* `Twitter` - https://twitter.com/BZEdgeCoin 
* `Telegram` - https://t.me/BZEdgeOfficial
* `Discord` - https://discord.gg/d73EvrNFvx
* `Github` https://github.com/bze-alphateam
* `Medium` https://medium.com/@bzedge
* `Official Shop` https://bzedge.shop/
### RPC
* `RPC` https://beezee.rpc.m.anode.team
### API
* `API` https://beezee.api.m.anode.team
### Peers and seeds
* `Peer` 246036506f4dc3905622a7c2d34553c321f21b7a@65.109.28.177:29577
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/bze-alphateam/bze/main/genesis.json
* `Addrbook` https://anode.team/BeeZee/main/addrbook.json
### Explorer
* `ANODE.TEAM` https://main.anode.team/beezee
* `BeeZee` https://explorer.getbze.com/
* `Ping` https://ping.pub/beezee
* `Atomscan` https://atomscan.com/beezee
## Installation Steps
>Prerequisite: go1.18.5+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/bze-alphateam/bze.git
git checkout v5.1.2
make build
```
* Init
```
bzed init <moniker> --chain-id beezee-1
bzed config chain-id beezee-1
```

### Generate keys
```
bzed keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/bze-alphateam/bze/main/genesis.json > ~/.bze/config/genesis.json
curl https://anode.team/BeeZee/main/addrbook.json > ~/.bze/config/addrbook.json
```
### Peers, seed
```
seeds="6385d5fb198e3a793498019bb8917973325e5eb7@51.15.228.169:26656,ce25088267cef31f3be1ec03263524764c5c80bb@163.172.130.162:26656,102d28592757192ccf709e7fbb08e7dd8721feb1@51.15.138.216:26656,f238198a75e886a21cd0522b6b06aa019b9e182e@51.15.55.142:26656,2624d40b8861415e004d4532bb7d8d90dd0e6e66@51.15.115.192:26656,d36f2bc75b0e7c28f6cd3cbd5bd50dc7ed8a0d11@38.242.227.150:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.bze/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/bzed.service > /dev/null <<EOF
[Unit]
Description=BeeZee Mainnet
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/
ExecStart=$(which bzed) start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable bzed
sudo systemctl restart bzed && journalctl -n 100 -f -u bzed
```
## Create Validator
```
bzed tx staking create-validator \
  --amount=3900000ubze \
  --pubkey=$(bzed tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id=beezee-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.1" \
  --min-self-delegation="1" \
  --gas="auto" \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://beezee.rpc.m.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop bzed && bzed unsafe-reset-all --home $HOME/.bze
```
```
peers="246036506f4dc3905622a7c2d34553c321f21b7a@65.109.28.177:29577"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.bze/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.bze/config/config.toml
```
```
sudo systemctl restart bzed && journalctl -n 100 -f -u bzed
```
