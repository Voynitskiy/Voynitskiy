# Nibiru [nibiru-testnet-1]
![Nibiru Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/testnet/Nibiru/Nibiru.png)
### Validator AlxVoy
* `Delegate` https://test.anode.team/nibiru/staking/nibivaloper1xt7q9pd4m3mp50kxzj9wsuu7t2xmlgvc66d5m3
* `Valoper` nibivaloper1xt7q9pd4m3mp50kxzj9wsuu7t2xmlgvc66d5m3
### Links
* `Website` - https://nibiru.fi/
* `Twitter` - https://twitter.com/NibiruChain
* `Discord` - https://discord.gg/nibiru
* `Github` https://github.com/NibiruChain
* `GitBook` https://docs.nibiru.fi/
* `Medium` https://blog.nibiru.fi/
* `LinkedIn` https://www.linkedin.com/company/nibiruchain/
### RPC
* `RPC` https://nibiru.rpc.t.anode.team
### API
* `API` https://nibiru.api.t.anode.team
### Peers and seeds
* `Peer` 5d9432668a2acd0587ecb77b5728177d216c02bc@65.109.93.152:36316
* `Live peers` https://anode.team/Nibiru/test/peers.txt
### Genesis and addrbook
* `Genesis` https://anode.team/Nibiru/test/genesis.json
* `Addrbook` https://anode.team/Nibiru/test/addrbook.json
### Explorer
* `ANODE.TEAM` https://test.anode.team/nibiru
### Auto installation script
```
wget https://anode.team/Nibiru/test/setup_nibiru.sh && chmod u+x setup_nibiru.sh && ./setup_nibiru.sh
```
## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/NibiruChain/nibiru
cd nibiru
git checkout v0.16.3
make install
```
* Init
```
nibid init <moniker> --chain-id nibiru-testnet-2
nibid config chain-id nibiru-testnet-2
```

### Generate keys
```
nibid keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://anode.team/Nibiru/test/genesis.json > ~/.nibid/config/genesis.json
curl https://anode.team/Nibiru/test/addrbook.json > ~/.nibid/config/addrbook.json
```
### Peers, seed
```
peers="5d9432668a2acd0587ecb77b5728177d216c02bc@65.109.93.152:36317"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.nibid/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/nibid.service > /dev/null <<EOF
[Unit]
Description=Nibiru
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nibid) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable nibid
sudo systemctl restart nibid && journalctl -fu nibid -o cat
```
## Create Validator
```
nibid tx staking create-validator \
  --amount=10000000unibi \
  --pubkey=$(nibid tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="nibiru-testnet-2" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --fees 5000unibi \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://nibiru.rpc.t.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop nibid && nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book
```
```
peers="5d9432668a2acd0587ecb77b5728177d216c02bc@65.109.93.152:36317"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.nibid/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.nibid/config/config.toml
```
```
curl -o - -L https://anode.team/Nibiru/test/anode.team_nibiru_wasm.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.nibid/data
```
```
sudo systemctl restart nibid && journalctl -fu nibid -o cat
```
