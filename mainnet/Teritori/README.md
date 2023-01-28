# Teritori [teritori-1]
![Teritori Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/Teritori/Teritori.png)
### Validator AlxVoy
* `Delegate` https://main.anode.team/teritori/staking/torivaloper1utr8j9685hfxyza3wnu8pa9lpu8360knpfvnq4
* `Valoper` torivaloper1utr8j9685hfxyza3wnu8pa9lpu8360knpfvnq4
### Links
* `Website` - https://teritori.com/
* `Twitter` - https://twitter.com/TeritoriNetwork
* `Discord` - https://discord.gg/teritori
* `CoinGecko` - https://www.coingecko.com/en/coins/teritori
* `Github` - https://github.com/TERITORI/
* `Crew3` - https://teritori.crew3.xyz/questboard
* `Medium` - https://medium.com/teritori
* `Whitepaper` - https://teritori.gitbook.io/teritori-whitepaper
* `Roadmap` - https://teritori.com/roadmap
* `Airdrop` - https://teritori.com/airdrop
* `The Alpha dApp` - https://app.teritori.com/
* `Marketplace` - https://app.teritori.com/marketplace
* `Launchpad Application Form` - https://airtable.com/shr1kU7kXW0267gNV

### The R!OT
* `Linktr` - https://linktr.ee/theriotnft
* `Discord` - https://discord.gg/theriotnft

### RPC
* `RPC` https://teritori.rpc.m.anode.team

### API
* `API` https://teritori.api.m.anode.team

### Peers
* `Peer` d2841ce68396c06b9793597feedbcef26ff87a8d@65.109.28.177:26796
* `Peer` https://anode.team/Teritori/main/peers.txt

### Genesis and addrbook
* `Genesis` https://anode.team/Teritori/main/genesis.json
* `Addrbook` https://anode.team/Teritori/main/addrbook.json

### Explorer
* `ANODE.TEAM` https://main.anode.team/teritori
* `Mintscan` https://www.mintscan.io/teritori
* `Ping` https://ping.pub/teritori
* `Atomscan` https://atomscan.com/teritori

### Auto installation script
```
wget https://anode.team/Teritori/main/setup_tori.sh && chmod u+x setup_tori.sh && ./setup_tori.sh
```

## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/TERITORI/teritori-chain
cd teritori-chain && git checkout v1.3.0
make install
```
* Init
```
teritorid init <moniker> --chain-id teritori-1
teritorid config chain-id teritori-1
```

### Generate keys
```
teritorid keys add <wallet_name>
```

### Genesis, addrbook
```
curl https://anode.team/Teritori/main/genesis.json > ~/.teritorid/config/genesis.json
curl https://anode.team/Teritori/main/addrbook.json > ~/.teritorid/config/addrbook.json
```

### Peers, seed
```
peers="3069b058b5ed85c3cdb2cf18fb1d255d966b53af@193.149.187.8:26656,a06fbbb9ace823ae28a696a91daa2d0644653c28@65.21.32.200:26756"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.teritorid/config/config.toml
```
### Add min gas
```
sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.001utori"/g' ~/.teritorid/config/app.toml
```

### Pruning
```
sed -i.bak -e "s/^pruning *=.*/pruning = \"nothing\"/" $HOME/.teritorid/config/app.toml
```

### Create the service file
```
sudo tee /etc/systemd/system/teritorid.service > /dev/null <<EOF
[Unit]
Description=Teritori Daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which teritorid) start
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable teritorid
sudo systemctl restart teritorid && journalctl -fu teritorid -o cat
```

## Create Validator
```
teritorid tx staking create-validator \
  --amount=1000000utori \
  --pubkey=$(teritorid tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="teritori-1" \
  --commission-rate="0.05" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1000000" \
  --from=<wallet_name>
```

### State-Sync
* start with State-Sync
```
SNAP_RPC=https://teritori.rpc.m.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop teritorid && teritorid tendermint unsafe-reset-all --home $HOME/.teritorid --keep-addr-book
```
```
peers="d2841ce68396c06b9793597feedbcef26ff87a8d@65.109.28.177:26796"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.teritorid/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.teritorid/config/config.toml
```
```
curl -o - -L https://anode.team/Teritori/main/anode.team_teritori_wasm.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.teritorid/data
```
```
sudo systemctl restart teritorid && journalctl -fu teritorid -o cat
```

### SnapShot (2 times a day)
```
sudo systemctl stop teritorid && \
cp $HOME/.teritorid/data/priv_validator_state.json $HOME/.teritorid/priv_validator_state.json.backup && \
rm -rf $HOME/.teritorid/data/
```
```
curl -o - -L https://anode.team/Teritori/main/anode.team_teritori.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.teritorid
```
```
mv $HOME/.teritorid/priv_validator_state.json.backup $HOME/.teritorid/data/priv_validator_state.json && \
sudo systemctl restart teritorid && journalctl -fu teritorid -o cat
```
