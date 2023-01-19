# Lava [lava-testnet-1]
![Lava Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/testnet/Lava/Lava.png)
### Validator AlxVoy
* `Delegate` https://test.anode.team/lava/staking/lava@valoper136r0tj6uzfq9hlnemng8gzz8qmhddahx3zw6sk
* `Valoper` lava@valoper136r0tj6uzfq9hlnemng8gzz8qmhddahx3zw6sk
### Links
* `Website` - https://lavanet.xyz/
* `Twitter` - https://twitter.com/lavanetxyz
* `Discord` - https://discord.gg/ZuuXGy6GNF
* `Medium` https://medium.com/lava-network
* `Litepaper` https://lavanet.xyz/assets/lava_litepaper_v0_1.pdf
* `Docs` https://docs.lavanet.xyz/
### RPC
* `RPC` https://lava.rpc.t.anode.team
### API
* `API` https://lava.api.t.anode.team
### Peers and seeds
* `Peer` f9190a58670c07f8202abfd9b5b14187b18d755b@144.76.97.251:27656
* `Live peers` https://anode.team/Lava/test/peers.txt
### Genesis and addrbook
* `Genesis` https://anode.team/Lava/test/genesis.json
* `Addrbook` https://anode.team/Lava/test/addrbook.json
### Explorer
* `ANODE.TEAM` https://test.anode.team/lava
### Auto installation script
```
wget https://anode.team/Lava/test/setup_lava.sh && chmod u+x setup_lava.sh && ./setup_lava.sh
```
## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
wget https://lava-binary-upgrades.s3.amazonaws.com/testnet/v0.3.0/lavad
chmod +x lavad
mv lavad $HOME/go/bin/
```
* Init
```
lavad init <moniker> --chain-id lava-testnet-1
lavad config chain-id lava-testnet-1
```

### Generate keys
```
lavad keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://anode.team/Lava/test/genesis.json > ~/.lava/config/genesis.json
curl https://anode.team/Lava/test/addrbook.json > ~/.lava/config/addrbook.json
```
### Peers, seed
```
seeds="3a445bfdbe2d0c8ee82461633aa3af31bc2b4dc0@prod-pnet-seed-node.lavanet.xyz:26656,e593c7a9ca61f5616119d6beb5bd8ef5dd28d62d@prod-pnet-seed-node2.lavanet.xyz:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.lava/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/lavad.service > /dev/null <<EOF
[Unit]
Description=Lava
After=network-online.target

[Service]
User=$USER
ExecStart=$(which lavad) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable lavad
sudo systemctl restart lavad && journalctl -fu lavad -o cat
```
## Create Validator
```
lavad tx staking create-validator \
  --amount=1000000000ulava \
  --pubkey=$(lavad tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="lava-testnet-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --fees="200ulava" \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://lava.rpc.t.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop lavad && lavad tendermint unsafe-reset-all --home $HOME/.lavad --keep-addr-book
```
```
peers="f9190a58670c07f8202abfd9b5b14187b18d755b@144.76.97.251:27656"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.lambdavm/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.lava/config/config.toml
```
```
sudo systemctl restart lavad && journalctl -u lavad -f
```
