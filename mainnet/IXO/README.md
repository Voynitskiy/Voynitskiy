# IXO [ixo-4]
![IXO Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/IXO/IXO.png)
### Validator AlxVoy
* `Delegate` https://main.anode.team/ixo/staking/ixovaloper1km58hq4m6kg8q2n3gxc84xt8v06fc24c4tc4wt
* `Valoper` ixovaloper1km58hq4m6kg8q2n3gxc84xt8v06fc24c4tc4wt
### Links
* `Website` - https://ixo.world/
* `Twitter` - https://twitter.com/ixoworld
* `Telegram` - https://t.me/ixonetwork
* `Github` - https://github.com/ixofoundation
* `Discord` - https://discord.gg/ixo
### RPC
* `RPC` https://ixo.rpc.m.anode.team
### API
* `API` https://ixo.api.m.anode.team
### Peers and seeds
* `Peer` 39526bf4668ef8a0919c11450372559b7315699a@144.76.97.251:26656
* `Live peers` https://anode.team/IXO/main/peers.txt
### Genesis and addrbook
* `Genesis` https://anode.team/IXO/main/genesis.json
* `Addrbook` https://anode.team/IXO/main/addrbook.json
### Explorer
* `ANODE.TEAM` https://main.anode.team/ixo
* `IXO` https://blockscan.ixo.world/
* `Mintscan` https://www.mintscan.io/ixo
* `Ping` https://ping.pub/ixo
* `Atomscan` https://atomscan.com/ixo
### Auto installation script
```
wget https://anode.team/IXO/main/setup_ixo.sh && chmod u+x setup_ixo.sh && ./setup_ixo.sh
```
## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/ixofoundation/ixo-blockchain.git
cd ixo-blockchain
git checkout v0.19.2
make install
```
* Init
```
ixod init <moniker> --chain-id ixo-4
```

### Generate keys
```
ixod keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://anode.team/IXO/main/genesis.json > ~/.ixod/config/genesis.json
curl https://anode.team/IXO/main/addrbook.json > ~/.ixod/config/addrbook.json
```
### Peers, seed
```
peers="a8d9811a2f08b8a6c77e4319097d6fd84520645e@139.84.226.60:26656,f79da5c87e40587c4cfef5d7b7902b6e69ac62bf@188.166.183.216:26656,386277f9c6a0c402889032ff76585d0a2dae7bc5@104.248.1.56:26656,26593e0854848ede80d5cd963dc8a775634e2acc@23.88.69.167:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.ixod/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/ixod.service > /dev/null <<EOF
[Unit]
Description=IXO Daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which ixod) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable ixod
sudo systemctl restart ixod && journalctl -fu ixod -o cat
```
## Create Validator
```
ixod tx staking create-validator \
  --amount=1000000uixo \
  --pubkey=$(ixod tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="ixo-4" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --node "tcp://127.0.0.1:26657" \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://ixo.rpc.m.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop ixod && ixod tendermint unsafe-reset-all --home $HOME/.ixod
```
```
peers="39526bf4668ef8a0919c11450372559b7315699a@144.76.97.251:26656"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.ixod/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.ixod/config/config.toml
```
```
sudo systemctl restart ixod && journalctl -fu ixod -o cat
```
